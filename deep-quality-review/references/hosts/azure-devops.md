# Azure DevOps

## Detect

A URL on `dev.azure.com` (`https://dev.azure.com/<org>/<project>/_git/<repo>/pullrequest/<N>`) or `<org>.visualstudio.com`, or a bare PR number in a repo whose `origin` remote is on one of those hosts. For a bare number, build the URL from the remote.

- `<slug>`: `<org>__<project>__<repo>`
- Requires `curl` and `python`. The token comes from `ADO_TOKEN`, otherwise from `git credential fill` for `dev.azure.com`, which works wherever `git fetch` from the repo already works. Posting needs a token with Code (read and write) scope.

Set these once per shell. Every later block uses them:

```bash
web="https://dev.azure.com/<org>/<project>/_git/<repo>"
api="https://dev.azure.com/<org>/<project>/_apis/git/repositories/<repo>/pullRequests/<N>"
v="api-version=7.1"
json="<tmp>/pr-reviews/<slug>-<N>-quality-review.json"
export ADO_TOKEN="${ADO_TOKEN:-$(printf 'protocol=https\nhost=dev.azure.com\npath=<org>/<project>/_git/<repo>\n\n' | GIT_TERMINAL_PROMPT=0 git credential fill | sed -n 's/^password=//p')}"
ado() { curl -sS --fail-with-body -u ":$ADO_TOKEN" -H "Content-Type: application/json" "$@"; }
```

A call that returns an HTML sign-in page instead of JSON means the token is missing or expired.

## Load the PR

```bash
ado "$api?$v" > "$json.pr"
ado "$api/workitems?$v"
```

From `$json.pr`:

| Field | Meaning |
|---|---|
| `title`, `description`, `createdBy.displayName` | title, description and author |
| `targetRefName` | base branch, after `refs/heads/` |
| `sourceRefName` | head ref |
| `lastMergeSourceCommit.commitId` | head SHA |
| `forkSource.repository.remoteUrl` | the fork to fetch from, when set |

The linked issues are the work items. When none is linked, look for an issue key in the title or the source branch name, such as `PROJ-123`. The changed files come from the diff in the worktree.

## Check out

- Clone: `git clone --filter=blob:none "$web" "$clone"`
- Head ref to fetch: `sourceRefName`. Azure DevOps has no ref for a PR's head, so after the fetch check that `refs/pr/<N>` is at the head SHA. If it isn't, the branch moved after the load, so load the PR again. When the PR comes from a fork, fetch `sourceRefName` from the fork's URL instead of `origin`.

## Comment ranges

Any range on the PR head of a file the PR changes works, and a whole-file comment works on any changed file. A file the PR doesn't change can't carry a comment, so move the item to a changed file or to the body.

## Line links

The payload cites lines as relative links, `[text](<path>#L<start>)` or `#L<start>-L<end>`. Before posting or editing, write a resolved copy of the payload, `$json.resolved`. It points each link at the line in the scored commit, in the form Azure DevOps builds when you select lines, and starts every body with the marker:

```bash
python - "$json" "$web" <<'EOF'
import json, re, sys, urllib.parse
path, web = sys.argv[1:3]
d = json.load(open(path, encoding="utf-8"))
mark = "<!-- deep-quality-review -->"

def link(m):
	start, end = int(m[2]), int(m[3] or m[2])
	query = f"path={urllib.parse.quote('/' + m[1].lstrip('/'))}&version=GC{d['commit_id']}&line={start}&lineEnd={end + 1}"
	return f"]({web}?{query}&lineStartColumn=1&lineEndColumn=1&lineStyle=plain&_a=contents)"

def resolve(body):
	return mark + "\n" + re.sub(r"\]\((?!https?:|#)([^)\s#]+)#L(\d+)(?:-L(\d+))?\)", link, body.replace(mark, "").strip())

d["body"] = resolve(d["body"])
for c in d["comments"] + d.get("file_comments", []):
	c["body"] = resolve(c["body"])
json.dump(d, open(path + ".resolved", "w", encoding="utf-8"))
EOF
```

## Open threads

This lists the skill's threads that are still open into `$json.open`, one JSON object per line: `thread` and first `comment` id, `kind` (`finding` or `summary`), `path`, `line` and `content`. Run it before resolving old threads and before editing a posted review:

```bash
ado "https://dev.azure.com/<org>/_apis/connectionData" > "$json.me"
ado "$api/threads?$v" > "$json.threads"
python - "$json" <<'EOF'
import json, sys
path = sys.argv[1]
me = json.load(open(path + ".me", encoding="utf-8"))["authenticatedUser"]["id"]
mark = "<!-- deep-quality-review -->"
with open(path + ".open", "w", encoding="utf-8") as out:
	for t in json.load(open(path + ".threads", encoding="utf-8"))["value"]:
		first = t["comments"][0]
		if first["author"]["id"] != me or first.get("isDeleted") or mark not in first.get("content", ""):
			continue
		context = t.get("threadContext") or {}
		kind = "finding" if context else "summary"
		if kind == "finding" and t.get("status") not in ("active", "pending"):
			continue
		row = {"thread": t["id"], "comment": first["id"], "kind": kind, "path": context.get("filePath"),
			"line": (context.get("rightFileStart") or {}).get("line"), "content": first["content"]}
		out.write(json.dumps(row) + "\n")
		print(row["thread"], kind, row["path"] or "", row["line"] or "")
EOF
```

## Earlier review threads

Every comment thread on the PR from any reviewer, whatever its status, with its path and line. System threads (votes, pushes) are left out:

```bash
ado "$api/threads?$v" | python -c "
import json,sys
for t in json.load(sys.stdin)['value']:
    cs=[c for c in t.get('comments',[]) if c.get('commentType')!='system' and not c.get('isDeleted')]
    if not cs: continue
    ctx=t.get('threadContext') or {}
    print(json.dumps({'status':t.get('status'),'path':ctx.get('filePath'),'line':(ctx.get('rightFileStart') or {}).get('line'),
        'comments':[{'author':c['author']['displayName'],'content':c.get('content','')} for c in cs]}))
"
```

## Edit a posted review

Edit the fixed comments and the body in the payload, write the resolved copy (see Line links) and list the open threads (see Open threads). Then rewrite each changed body. Each item is matched to its newest open thread by path and first line:

```bash
python - "$json" <<'EOF' | while read -r thread comment file; do ado -X PATCH --data-binary "@$file" "$api/threads/$thread/comments/$comment?$v" > /dev/null && echo "updated $thread"; done
import json, sys
path = sys.argv[1]
d = json.load(open(path + ".resolved", encoding="utf-8"))
newest = {}
for line in open(path + ".open", encoding="utf-8"):
	row = json.loads(line)
	key = "summary" if row["kind"] == "summary" else (row["path"], row["line"])
	if key not in newest or row["thread"] > newest[key]["thread"]:
		newest[key] = row
items = [(("/" + c["path"].lstrip("/"), c.get("start_line", c["line"])), c["body"]) for c in d["comments"]]
items += [(("/" + c["path"].lstrip("/"), None), c["body"]) for c in d.get("file_comments", [])]
items.append(("summary", d["body"]))
for i, (key, body) in enumerate(items):
	row = newest.get(key)
	if row is None:
		print(f"No open thread for {key}, skipped.", file=sys.stderr)
	elif body != row["content"]:
		patch = f"{path}.patch{i:02}"
		json.dump({"content": body}, open(patch, "w", encoding="utf-8"))
		print(row["thread"], row["comment"], patch)
EOF
```

To drop a posted comment, delete it with `ado -X DELETE "$api/threads/<thread>/comments/<comment>?$v"` and remove it from the payload. A posted comment can't be moved to another line.

## Post

Azure DevOps has no review object, so the review is a set of threads, each tagged with the marker:

- each line-range and whole-file comment is its own thread, posted `active`
- the body is one general thread, posted `closed`, so the scorecard is visible but never counts as an open comment
- the skill never votes on the PR, except with `--gate` (see step 3)

When the target branch has a comment-resolution policy, the `active` threads hold completion until someone resolves them, the same as any reviewer's comments. Before posting, tell the user how many active threads the review adds.

### 1. Resolve old review threads

List the open threads. For each `finding` thread whose finding the author addressed, set it to fixed with `ado -X PATCH --data '{"status": 2}' "$api/threads/<thread>?$v"`. For each `summary` thread, PATCH its comment to the same content with the marker removed, so the next run doesn't find it again. Finding threads left active are checked again on the next run.

### 2. Post the review

Write the resolved copy (see Line links). Check that the PR's latest iteration is still at the payload's `commit_id`, and fetch the changed files' tracking ids:
```bash
read -r iteration sha < <(ado "$api/iterations?$v" | python -c "import json,sys; i=json.load(sys.stdin)['value'][-1]; print(i['id'], i['sourceRefCommit']['commitId'])")
ado "$api/iterations/$iteration/changes?\$top=2000&$v" > "$json.changes"
```
If `sha` isn't the payload's `commit_id`, the PR has new commits, so rerun the review instead of posting. If `$json.changes` has a non-zero `nextSkip`, fetch the rest with `&\$skip=<nextSkip>` and add their `changeEntries`.

Build one thread file per comment, with the scorecard last, then post them in order. The first line clears an earlier run's thread files:
```bash
rm -f "$json".thread??
python - "$json" "<worktree>" "$iteration" <<'EOF'
import json, os, sys
path, worktree, iteration = sys.argv[1], sys.argv[2], int(sys.argv[3])
d = json.load(open(path + ".resolved", encoding="utf-8"))
changes = json.load(open(path + ".changes", encoding="utf-8"))["changeEntries"]
tracking = {e["item"]["path"]: e["changeTrackingId"] for e in changes if "path" in e["item"]}

def context(file):
	server = "/" + file.lstrip("/")
	if server not in tracking:
		sys.exit(f"{file} is not changed in the PR. Move the comment to a file the PR changes.")
	iterations = {"firstComparingIteration": iteration, "secondComparingIteration": iteration}
	return server, {"changeTrackingId": tracking[server], "iterationContext": iterations}

def thread(body, status, where=None, pr_context=None):
	t = {"comments": [{"parentCommentId": 0, "content": body, "commentType": 1}], "status": status}
	if where:
		t["threadContext"], t["pullRequestThreadContext"] = where, pr_context
	return t

threads = []
for c in d["comments"]:
	server, pr_context = context(c["path"])
	lines = open(os.path.join(worktree, c["path"]), encoding="utf-8").read().splitlines()
	end = c["line"]
	where = {"filePath": server, "rightFileStart": {"line": c.get("start_line", end), "offset": 1},
		"rightFileEnd": {"line": end, "offset": len(lines[end - 1]) + 1}}
	threads.append(thread(c["body"], 1, where, pr_context))
for c in d.get("file_comments", []):
	server, pr_context = context(c["path"])
	threads.append(thread(c["body"], 1, {"filePath": server}, pr_context))
threads.append(thread(d["body"], 4))
for i, t in enumerate(threads):
	json.dump(t, open(f"{path}.thread{i:02}", "w", encoding="utf-8"))
EOF
ok=1
for f in "$json".thread??; do
	id=$(ado -X POST --data-binary "@$f" "$api/threads?$v" | python -c "import json,sys; print(json.load(sys.stdin)['id'])") || { ok=0; echo "Failed to post $f." >&2; break; }
	echo "posted $id"
done
[ "$ok" = 1 ] && echo "$web/pullrequest/<N>?discussionId=$id"
```
If a post fails partway, the loop stops and prints no review URL. List the open threads, delete the comments this run posted, fix the cause and post again.

On success, the review URL is the last line printed, the scorecard thread.

### 3. Vote (`--gate` only)

Once every thread is posted, cast the gate event as your vote: `APPROVE` is `10` (approved) and `REQUEST_CHANGES` is `-5` (waiting for author):

```bash
me=$(ado "https://dev.azure.com/<org>/_apis/connectionData" | python -c "import json,sys; print(json.load(sys.stdin)['authenticatedUser']['id'])")
ado -X PUT --data '{"vote": <10 or -5>}' "$api/reviewers/$me?$v" > /dev/null && echo "voted"
```

If the vote fails, report the error. The threads stay posted, but the review isn't a gate until the vote lands.
