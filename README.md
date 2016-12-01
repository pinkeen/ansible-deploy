# Ansible deploy

A capistrano-style deploy for your web app.

## Assumptions and requirements

It should run on any linux target.

The assumption is made that the application is owned by a specific user, which 
should be an "interactive" user. System users like httpd may not work because
of how rsync works, you may not also be able to execute sudo commands.

The whole role shall be ran as the target (owner) user, meaning the the user which you use
to ssh into the target. `become_user` should also work but is not tested yet.
 
The target directory (`deploy_project_home`) must exist prior to running this role. Also it's
owner has to be the target user.

## Example (minimal) playbook

```yml
- name: Deploy App
  hosts: app_servers
  user: your_app_user
  roles:
    - role: pinkeen.deploy
      deploy_project_home: /var/www/your_app_user
      deploy_method: rsync
      deploy_artifact_src_dir: build/
```

## Variables

### `deploy_project_home: (required)`

The home directory of the application. Might be the app user's home directory
but does not have to be.
 
Do not make it a `/home/user` directory. Webserver will not be able to read the files
because of `/home` permissions in linux.

The user and the directory must exist prior to running the role. It's owner must be
the user.

### `deploy_releases_dir: "{{ deploy_project_home }}/releases"`

Directory where releases are kept. 

### `deploy_shared_dir: "{{ deploy_project_home }}/shared"`

Directory where directories shared between releases go. 

### `deploy_current_release_dir: "{{ deploy_project_home }}/current"`

Current release symlink.

### `deploy_previous_release_dir: "{{ deploy_project_home }}/previous"`

Previous release symlink,

### `deploy_method: git`

There are two deploy methods `git` and `rsync`. 

The `git` methods pulls the project on the remote machine. The correct ssh
configuration (key) must be provided before the role is executed.

### `deploy_git_repo: (required if deploy_method == 'git')`

Used with `git` deploy method. URI of the target repo.

### `deploy_git_branch: master`

Used with `git` deploy method. Branch that should be cloned.

### `deploy_artifact_src_dir: (required if deploy_method == 'rsync')`

Local directory which contains the artifact to be copied using rsync.

_Beware! Use a trailing slash or the rsync will recreate the part of the path
on the target machine._

### `deploy_copy_previous_release: True`

Used with `rsync` deploy method. If true then previous release (if available) is
copied to speed up the rsync - assuming that most of the files do not change between
releases.

### `deploy_rsync_opts: []`

Directly passed to `rsync_opts` of `synchronize` action. Useful excluding files
which shall be transfered.

### `deploy_keep_previous_releases: 3`

How many releases to keep at any time.

### `deploy_shared_dirs: []`

Directories shared between releases. They will be symlinked to a subdirectory of
`deploy_shared_dir`. This will be usually something like your logs, uploads...

The directories names shall be relative to the app directory.

### `deploy_config_files: []`

Template files that should be rendered and copied into the app.

Each item should have two keys: `src` - the source file relative to playbook dir
and `dest` - target file relative to application dir.

### `deploy_server_writable_dirs: []`

Directories which will be made writable for the webserver.

### `deploy_use_acl: True`

If true then ACL permissions are used for setting up the writable dirs.

It's highly encouraged to leave this at `True` if possible.

If set to `False`, then the target user is added to the webserver group.
The writable dirs get rwx permissions (with group bit set) for this group.
Your application has to use `002` mask if you don't use ACL. 

### `deploy_server_user: (guessed)`

The user/group which is used by your webserver. If no value is provided then the
role tries to guess it by listing running processes.

### `deploy_app_user: (guessed)`

The target user owning the app. 

### `deploy_post_install_commands: []`

The commands that shall be executed after full install but before the directory
is switched into production.

The commands are executed in the application (current release) directory.

Typically this will be stuff like clearing cache, running migrations...

### `deploy_post_deploy_commands: []`

Commands that are ran after the app has been installed and the directory (symlink)
switched to live.

Typically you will want to reload the webserver, supervisord, ... here.

Note that you are running the role as a normal user!

If your commands require root then you have to take care of that. The best way
would be to use sudo and add special sudoers entries for this user.