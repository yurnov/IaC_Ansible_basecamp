## Introduction to Jinja2 

Jinja is a modern and designer-friendly templating language for Python, modelled after Django’s templates. It is fast, widely used and secure with the optional sandboxed template execution environment:

```
<title>{% block title %}{% endblock %}</title>
<ul>
{% for user in users %}
  <li><a href="{{ user.url }}">{{ user.username }}</a></li>
{% endfor %}
</ul>
```

Jinja2 used videley for variables and well as by `template` module to create configuration and files based on Jinja2 templates. 

Example:

```
- name: Template a file to /etc/file.conf
  template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
    owner: bin
    group: wheel
    mode: '0644'
```

Example of template content:

```
server {
  listen       80;
  server_name  localhost;
  location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
  }

{% if server_name is defined %}
server {
  listen 443 ssl;
  ssl_certificate /etc/nginx/conf.d/cert/fullchain.pem;
  ssl_certificate_key /etc/nginx/conf.d/cert/privkey.pem;
  include /etc/nginx/conf.d/cert/le.conf;
  ssl_dhparam /etc/nginx/conf.d/cert/ssl-dhparams.pem;

  server_name "{{ server_name }}";

  location = / {
    return 301 /"{{ subfolder }}"/;
  }

 location / {
    proxy_pass "{{ backend_address }}";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_set_header X-Forwarded-Protocol $scheme;
  }
}

server {
    listen 80;
    server_name "{{ server_name }}";
    return 301 https://$host$request_uri;
}
{% endif %}

```

## Future reading

[Jinja 2.11.x](https://jinja.palletsprojects.com/en/2.11.x/)
[Ansible template built-in module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html)
[Templating (Jinja2)](https://docs.ansible.com/ansible/latest/user_guide/playbooks_templating.html)
