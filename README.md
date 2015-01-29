# Copyright (c) 2007 Andy Parkins
#
# An example hook script to mail out commit update information.
#
# NOTE: This script is no longer under active development. There
# is another script, git-multimail, which is more capable and
# configurable and is largely backwards-compatible with this script;
# please see "contrib/hooks/multimail/". For instructions on how to
# migrate from post-receive-email to git-multimail, please see
# "README.migrate-from-post-receive-email" in that directory.
#
# This hook sends emails listing new revisions to the repository
# introduced by the change being reported. The rule is that (for
# branch updates) each commit will appear on one email and one email
# only.
#
# This hook is stored in the contrib/hooks directory. Your distribution
# will have put this somewhere standard. You should make this script
# executable then link to it in the repository you would like to use it in.
# For example, on debian the hook is stored in
# /usr/share/git-core/contrib/hooks/post-receive-email:
#
# cd /path/to/your/repository.git
# ln -sf /usr/share/git-core/contrib/hooks/post-receive-email hooks/post-receive
#
# This hook script assumes it is enabled on the central repository of a
# project, with all users pushing only to it and not between each other. It
# will still work if you don't operate in that style, but it would become
# possible for the email to be from someone other than the person doing the
# push.
#
# To help with debugging and use on pre-v1.5.1 git servers, this script will
# also obey the interface of hooks/update, taking its arguments on the
# command line. Unfortunately, hooks/update is called once for each ref.
# To avoid firing one email per ref, this script just prints its output to
# the screen when used in this mode. The output can then be redirected if
# wanted.
#
# Config
# ------
# hooks.mailinglist
# This is the list that all pushes will go to; leave it blank to not send
# emails for every ref update.
# hooks.announcelist
# This is the list that all pushes of annotated tags will go to. Leave it
# blank to default to the mailinglist field. The announce emails lists
# the short log summary of the changes since the last annotated tag.
# hooks.envelopesender
# If set then the -f option is passed to sendmail to allow the envelope
# sender address to be set
# hooks.emailprefix
# All emails have their subjects prefixed with this prefix, or "[SCM]"
# if emailprefix is unset, to aid filtering
# hooks.showrev
# The shell command used to format each revision in the email, with
# "%s" replaced with the commit id. Defaults to "git rev-list -1
# --pretty %s", displaying the commit id, author, date and log
# message. To list full patches separated by a blank line, you
# could set this to "git show -C %s; echo".
# To list a gitweb/cgit URL *and* a full patch for each change set, use this:
# "t=%s; printf 'http://.../?id=%%s' \$t; echo;echo; git show -C \$t; echo"
# Be careful if "..." contains things that will be expanded by shell "eval"
# or printf.
# hooks.emailmaxlines
# The maximum number of lines that should be included in the generated
# email body. If not specified, there is no limit.
# Lines beyond the limit are suppressed and counted, and a final
# line is added indicating the number of suppressed lines.
# hooks.diffopts
# Alternate options for the git diff-tree invocation that shows changes.
# Default is "--stat --summary --find-copies-harder". Add -p to those
# options to include a unified diff of changes in addition to the usual
# summary output.
#
# Notes
# -----
# All emails include the headers "X-Git-Refname", "X-Git-Oldrev",
# "X-Git-Newrev", and "X-Git-Reftype" to enable fine tuned filtering and
# give information for debugging.
#
# ---------------------------- Functions
#
# Function to prepare for email generation. This decides what type
# of update this is and whether an email should even be generated.
#
