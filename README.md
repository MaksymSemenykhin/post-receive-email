# Copyright (c) 2007 Andy Parkins

 This hook sends emails listing new revisions to the repository
 introduced by the change being reported. The rule is that (for
 branch updates) each commit will appear on one email and one email
 only.

 This hook is stored in the contrib/hooks directory. Your distribution
 will have put this somewhere standard. You should make this script
 executable then link to it in the repository you would like to use it in.
 For example, on debian the hook is stored in
 /usr/share/git-core/contrib/hooks/post-receive-email:

# .git/Config
[hooks]
mailinglist = "msem@*.ua, syam@*.ua"
envelopesender = release@*.ua
emailprefix = "[GIT] "
