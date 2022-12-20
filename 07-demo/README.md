Create enviroment variable `DIGITALOCEAN_TOKEN` and than build and run container:

```
docker build . -t basecamp-demo
docker run -it --rm -e DIGITALOCEAN_TOKEN=${DIGITALOCEAN_TOKEN} -v ~/.ssh:/home/ansible/.ssh basecamp-demo
```

Then inside container execute:

```
ansible-playbook -e DIGITALOCEAN_TOKEN=${DIGITALOCEAN_TOKEN} create_and_configure_droplet.yml
```