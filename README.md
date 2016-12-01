# Ansible deploy

deploy_project_home: (required)
deploy_releases_dir: "{{ deploy_project_home }}/releases"
deploy_shared_dir: "{{ deploy_project_home }}/shared"
deploy_current_release_dir: "{{ deploy_project_home }}/current"
deploy_previous_release_dir: "{{ deploy_project_home }}/previous"
deploy_copy_previous_release: True
deploy_method: git
deploy_git_repo: (required if deploy_method == 'git')
deploy_git_branch: master
deploy_keep_previous_releases: 3
deploy_shared_dirs: []
deploy_config_files: []
deploy_rsync_opts: []
deploy_artifact_src_dir: (required if deploy_method == 'rsync')
deploy_server_writable_dirs: []
deploy_server_user: (guessed)
deploy_app_user: (guessed)
deploy_use_acl: True