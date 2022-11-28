To run docker container:

```
docker run -it -v ~/.ssh:/home/ansible/.ssh (image hash) bash
```

Then inside container execute:

```
cd /opt/
ansible-playbook -e DIGITALOCEAN_TOKEN=${DIGITALOCEAN_TOKEN} create_and_configure_droplet.yml
```