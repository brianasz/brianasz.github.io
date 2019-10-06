---
layout: post
title: Using fastly to block brute force attacks
---

attacked URL: /uk/en/login/authenticate
Pattern in attack: POST methods without HTTP Session headers, for example (Session-Id)
Block in fastly:
- name: block auth posts without session
  comment: ''
  priority: '10'
  statement: req.url ~ "/(uk|us)/en/login/authenticate" && req.method == "POST" &&
    req.http.Cookie !~ "SESSION="
  type: REQUEST