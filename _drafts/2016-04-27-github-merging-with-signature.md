---
layout: post
title:  "GitHub Merging with Signature"
date:   2016-04-27 04:07:28
categories: github
---

Well, this was an interesting adventure.  I recently began [signing my commits](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work) for no particular reason other than to add some reputation to my public key - everything was pretty easy to set up since I already had a [Yubikey](https://www.yubico.com/products/yubikey-hardware/) configured and since GitHub [recently implemented visuals for this](https://github.com/blog/2144-gpg-signature-verification).  Once the necessary libraries were set up and working properly and I had my git config set to auto-sign all commits, everything looked perfect.

Until I merged a pull request.

As you'd expect, performing `git merge --no-ff` creates a commit, which is exactly what clicking the "Merge Pull Request" button does - and when you're making a commit via the webui, you obviously don't have access to the resources required to sign this commit.  Finding out how to do that via the CLI proved a little more tricky than I expected, though.  You could be satisfied by expanding the field near the PR merge button which instructs you to create a new local branch, checkout the candidate into that branch, then perform the signed merge manually, which does work assuming you know exactly where the merge is coming from.  But what if you don't want to pull up the webui and dig for where the PR came from specifically?  What if that resource became stale and was deleted?

### Enter obscurely-documented features.

At this point, I got curious as to what a pull request actually is.  I ran into [this SO answer](http://stackoverflow.com/a/24090258) to a question which appeared identical to mine - but the [article linked](https://github.com/git/git/blob/master/Documentation/howto/using-signed-tag-in-pull-request.txt) seemed to imply this relied on creating signed tags and exporting a `git request-pull` which is clearly _not_ what a GitHub Pull Request is.  (Incidentally, one of the other linked sources was actually a blatant modification of that original email thread.)  After some digging, I found [this page on GitHub Help](https://help.github.com/articles/checking-out-pull-requests-locally/) which reluctantly details how to merge a PR in the unlikely event the original owner has deleted their fork: `git fetch origin pull/ID/head:BRANCHNAME` where ID is the Pull Request ID# and BRANCHNAME is a temporary branch to fetch the PR into.  Okay, got it: `fetch`, `merge --no-ff`, `push`.

Hmmm. Something's missing here? If I pass `-m 'comment'` to `merge`, I change the title of this merge but get no expanded description.  Possibly out of inexperience, the existence of the `-e` flag was new to me and thus I learned how to add some expanded comments through vim during the merge process.  As it turns out, an out of date vim that's shipped with OSX has some [inconsistencies when called as vi](https://github.com/VundleVim/Vundle.vim/issues/167#issuecomment-55700048) which is solvable by setting `git config --global core.editor $(which vim)`.  Finally, signed merges that look just like the webui does - though if [protected branch status checks](https://help.github.com/articles/about-required-status-checks/) are implemented, you obviously still have to make sure those pass manually, else you're going to merge potentially broken code to your upstream branch.

For easy reference, here's the whole workflow:

```bash
# assume signing is enabled globally, else add -s
# clone source for the PR with ID manually or use
git fetch origin pull/ID/head:temp
git checkout master
git merge --no-ff -m 'Merge pull request #ID from <source>' -e temp
git push
```

## Bonus points

If you happen to be working with many PRs and wish to have those readily accessible, [this gist](https://gist.github.com/piscisaureus/3342247) should help significantly. The suggested snippet of mending `git fetch` to add `refs/pull/*/head:refs/remotes/origin/pr/*` maps all pull requests to local branches at pr/ID, making branch switching and merging very easy.  Note that applying this alias globally is _potentially dangerous_, breaking things like the Homebrew install script or creating undeletable remotes named `origin` in every local repo.