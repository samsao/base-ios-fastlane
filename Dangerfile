# Sometimes it's a README fix, or something like that - which isn't relevant for
# including in a project's CHANGELOG for example
declared_trivial = github.pr_title.include? "#trivial"

# Make it more obvious that a PR is a work in progress and shouldn't be merged yet
warn("PR is classed as Work in Progress: DO NOT MERGE") if github.pr_title.include? "[WIP]"

# Warn when there is a big PR
warn("Big PR") if git.lines_of_code > 500

# Warn when a PR has no description
warn("Please provide a summary in the PR description") if github.pr_body.length < 5

# Makes sure only PRs to develop are created
fail "Please re-submit this PR to develop." if github.branch_for_base != "develop"


# Don't let testing shortcuts get into master by accident
fail("fdescribe left in tests") if `grep -r fdescribe specs/ `.length > 1
fail("fit left in tests") if `grep -r fit specs/ `.length > 1

# Checks spelling
prose.check_spelling
prose.ignored_words = ["kenza", "robin", "samsao", "yqb"]

# Reports on test coverage - doesnt work ATM
xcov.report(
	scheme: 'YQB',
	workspace: 'YQB.xcworkspace',
	configuration: 'Debug'
	)

# To test locally, run: bundle exec danger local
