# GitHub

## Detect

A URL on `github.com` (`https://github.com/<owner>/<repo>/pull/<N>`), `owner/repo#N`, or a bare PR number in a repo whose `origin` remote is on `github.com`.

- `<slug>`: `<owner>__<repo>`
- Requires `gh`, authenticated with access to the repo, and `python` for the posting step.

## Load the PR

```bash
gh pr view <N> -R <owner/repo> --json number,url,title,body,author,baseRefName,headRefName,headRefOid,files,additions,deletions
```

The head SHA is `headRefOid`. The base branch is `baseRefName`. Linked issues are the `closingIssuesReferences` field, or issue links in the body.

## Check out

- Clone: `gh repo clone <owner/repo> "$clone" -- --filter=blob:none`
- Head ref to fetch: `pull/<N>/head`

## Comment ranges

GitHub only accepts a range on the PR head inside one diff hunk. Check with:

```bash
gh api repos/<owner>/<repo>/pulls/<N>/files --paginate --jq '.[] | .filename, .patch'
```

An added file is one hunk, so any range in it works. A modified file needs both ends inside the same `@@ +start,count @@` hunk. Trim a range that crosses a hunk edge to the hunk that holds the change. A whole-file comment works on any file the PR touches.

## Line links

The payload cites lines as relative links, `[text](<path>#L<start>)` or `#L<start>-L<end>`. Before posting or editing a body, point each one at the scored commit, `https://github.com/<owner>/<repo>/blob/<commit_id>/<path>#L<start>-L<end>`. The Post step's first script does this for the whole payload.

## Edit a posted review

Resolve the line links in each new body first (see Line links). Apply the audit with `PATCH repos/<owner>/<repo>/pulls/comments/<id>` for fixed comments, `DELETE` on the same URL for dropped ones, and `PUT repos/<owner>/<repo>/pulls/<N>/reviews/<review id>` for the body. A posted comment can't be moved to another line.

## Post

The review is posted as a single `COMMENT` review, never `APPROVE` or `REQUEST_CHANGES`.

Find the earlier quality reviews, if any. There can be more than one, because a follow-up review carries the same marker:

```bash
me=$(gh api user --jq .login)
R="repos/<owner>/<repo>"
old=$(gh api "$R/pulls/<N>/reviews" --paginate \
  --jq ".[] | select(.user.login == \"$me\" and (.body | contains(\"<!-- deep-quality-review -->\"))) | .id")
```

1. **Resolve old review comments.** For every comment in the previous review, resolve it if the author addressed the finding. Once every comment in a previous review is resolved, update its body to drop the `<!-- deep-quality-review -->` marker, so the next run doesn't find it again. A review with unresolved comments keeps its marker, so the next run checks them again.

2. **Post the review** in three calls. The REST review endpoint takes line ranges but not whole-file comments, and GraphQL's `addPullRequestReviewThread` takes those only on a pending review. The first call creates the review pending, because the payload has no `event`:
   ```bash
   json=<review json>
   # Resolve the line links, then create the pending review with the line-range comments, on the right side of the diff.
   python -c "
   import json,re,sys
   d=json.load(open(sys.argv[1],encoding='utf-8'))
   blob=f'https://github.com/{sys.argv[2]}/blob/{d[\"commit_id\"]}/'
   link=lambda m: f'](' + blob + m[1].lstrip('/') + f'#L{m[2]}' + (f'-L{m[3]}' if m[3] else '') + ')'
   fix=lambda t: re.sub(r'\]\((?!https?:|#)([^)\s#]+)#L(\d+)(?:-L(\d+))?\)', link, t)
   d['body']=fix(d['body'])
   for c in d['comments']+d.get('file_comments',[]): c['body']=fix(c['body'])
   json.dump(d,open(sys.argv[1]+'.resolved','w',encoding='utf-8'))
   d.pop('file_comments',None)
   for c in d['comments']:
       c['side']='RIGHT'
       if 'start_line' in c: c['start_side']='RIGHT'
   json.dump(d,open(sys.argv[1]+'.post','w',encoding='utf-8'))
   " "$json" "<owner>/<repo>"
   json_resolved="$json.resolved"
   read rid node < <(gh api "$R/pulls/<N>/reviews" --method POST --input "$json.post" --jq '"\(.id) \(.node_id)"')
   # Add each whole-file comment, then submit as COMMENT with the body. Clear an earlier run's files first.
   rm -f "$json_resolved".file*
   python -c "
   import json,sys
   d=json.load(open(sys.argv[1],encoding='utf-8'))
   q='mutation(\$r:ID!,\$p:String!,\$b:String!){addPullRequestReviewThread(input:{pullRequestReviewId:\$r,path:\$p,body:\$b,subjectType:FILE}){thread{id}}}'
   for i,c in enumerate(d.get('file_comments',[])):
       json.dump({'query':q,'variables':{'r':sys.argv[2],'p':c['path'],'b':c['body']}},open(f'{sys.argv[1]}.file{i}','w',encoding='utf-8'))
   json.dump({'event':'COMMENT','body':d['body']},open(sys.argv[1]+'.submit','w',encoding='utf-8'))
   " "$json_resolved" "$node"
   ok=1
   for f in "$json_resolved".file*; do
       [ -e "$f" ] || continue
       gh api graphql --input "$f" --jq .data.addPullRequestReviewThread.thread.id || { ok=0; break; }
   done
   if [ "$ok" = 1 ]; then
       gh api "$R/pulls/<N>/reviews/$rid/events" --method POST --input "$json_resolved.submit" --jq .html_url
   else
       gh api "$R/pulls/<N>/reviews/$rid" --method DELETE > /dev/null
       echo "A whole-file comment failed, so the pending review was deleted." >&2
   fi
   ```
   If the first call fails with a 422 on a range, fix the range (see Comment ranges) and post again. A failed whole-file comment deletes the pending review, so fix the cause and post again. If the submit call fails, delete the pending review (`gh api "$R/pulls/<N>/reviews/$rid" --method DELETE`) before retrying, because GitHub allows one pending review per user per PR.

The review URL is the last call's output.
