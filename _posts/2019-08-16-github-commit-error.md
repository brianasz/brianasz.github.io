Errod:

brianasz@brianasz-ThinkPad-X1-Carbon-4th:~/brianaszBlog/brianasz.github.io$ git commit -m "Updating blog and adding new post (haproxy post)"

*** Please tell me who you are.

Run

  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"

to set your account's default identity.
Omit --global to set the identity only in this repository.

fatal: unable to auto-detect email address (got 'brianasz@thinkPad.(none)')


This didn't help:
git config --global user.mail brianasz@protonmail.com
git config --global user.name brianasz


Solution:
Tuve que poner el hostname de la máquina a protonmail.com

Terminal promtp ahora es:
brianasz@protonmail.com