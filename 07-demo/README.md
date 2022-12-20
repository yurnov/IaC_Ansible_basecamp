To run docker container create enviroment variable `DIGITALOCEAN_TOKEN` with your token and execute:

```
docker run -it -e DIGITALOCEAN_TOKEN=${DIGITALOCEAN_TOKEN} -v ~/.ssh:/home/ansible/.ssh (image hash) bash
```

Then inside container execute:

```
cd /opt/
ansible-playbook -e DIGITALOCEAN_TOKEN=${DIGITALOCEAN_TOKEN} create_and_configure_droplet.yml
```