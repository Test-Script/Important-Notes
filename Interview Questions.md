Q. Difference Between Git pull, Git Fetch and Git clone.

Git Clone : Copy the content into local workspace from the remote repository.

Git pull : Download the changes from the remote repository & also merging with local working directory.

Ex. git pull <Remote_Name> <branch-name>

$ git pull origin main
From https://github.com/Test-Script/Notes
 * branch            main       -> FETCH_HEAD
Updating 98b1243..f9cce6d
Fast-forward
 Linux GUI Remove.md => RHEL GUI Mode Initiating Procedure.md | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
 rename Linux GUI Remove.md => RHEL GUI Mode Initiating Procedure.md (98%)

Git Fetch : Download the changes from the remote Repository

Git remote -v : Shows the remote URL paths like fetch & push