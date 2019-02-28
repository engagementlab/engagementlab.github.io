# Engagement Lab's MEAN Stack

[![Lab logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/logo-bootstrapper.png "Engagement Lab logo")](http://elab.emerson.edu/)
[![MEAN logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/mean.png "Mean logo")](http://mean.io/)
[![Mongo logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/mongo.png "Mongodb logo")](http://mongodb.com/)
[![Express logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/express.png "Express logo")](https://expressjs.com/)
[![Angular logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/angular.png "Angular logo")](https://angular.io/)
[![Node logo](https://res.cloudinary.com/engagement-lab-home/image/upload/f_auto,c_scale,w_100//logos/node-sm.png "Node logo")](https://nodejs.org/)


## Getting Started

Almost all of the Engagement Lab's web projects use the [Mean](http://mean.io/) technology stack. You must be adequately familiar with it in order to be able to run and modify such projects. This guide does not go over in-depth this tech stack, as help is widely and freely available. [This tutorial on Heroku's Dev Center](https://devcenter.heroku.com/articles/mean-apps-restful-api) is a good place to begin.

### Prerequisites

Install [MongoDB](https://docs.mongodb.com/manual/administration/install-community/) and Node (via nvm, see below).

---
Optimally, you ought to [install nvm](https://github.com/creationix/nvm/blob/master/README.md) on the machine where you're running a project, such that you're running each project under its tested Node version. (You may need to install [wget](https://www.gnu.org/software/wget/faq.html#download) in order to get nvm.)

----
Open bash profile (if it doesn't exist, run ```touch ~/.bash_profile```):
```
nano ~/.bash_profile
```
Paste these lines in to have nvm and grunt automatically sourced upon login:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
```

Then, restart your terminal.
----
_Please read our [bootstrapper docs](https://www.npmjs.com/package/@engagementlab/el-bootstrapper) for necessary steps for server component configuration._

## Development server

Most of our MEAN projects are seperated into two major components: `client/` and `server/`.

- `client/` contains the Angular projects and constitutes the front-end of the app.
- `server/` contains the [express](https://expressjs.com/) server which runs the project's data API, and the KeystoneJS-based CMS via our [bootstrapper](https://www.npmjs.com/package/@engagementlab/el-bootstrapper) module. This component is primarily a [basic](https://expressjs.com/en/guide/routing.html) express app.

You need to run `npm i -g npm-run-all` and `npm i -g nodemon` in order to properly run the dev app.

In the client directory, run `nvm use && npm start` for a dev server and client. 

You may be prompted to install the relevant node version via `nvm install` when running a project if you don't have it yet.

Navigate to `http://localhost:4200/` for client app. The server is running at `http://localhost:3000` by default. Both the client and server will automatically reload if you change any of the source files. 

You can enter the CMS via `http://localhost:3000/keystone`; if you configured [bootstrapper](https://www.npmjs.com/package/@engagementlab/el-bootstrapper) correctly, you don't have to login on dev.

# Building and running production server
## Pre-requisites

As with the development server, you will need [MongoDB](https://mongodb.org) installed. [nvm](https://nvm.sh) is recommended but optional.

You will also need [nginx](http://nginx.org/en/download.html) and [pm2](https://pm2.io/runtime/).

## Reverse proxy

As noted in express's "[Production best practices](http://expressjs.com/en/advanced/best-practice-performance.html)", it is a good idea generally to run an express-based app begin a reverse proxy. Handing over tasks that do not require knowledge of application state to a reverse proxy frees up Express to perform specialized application tasks.

I highly recommend using [nginx](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/) since it's lightweight and absurdly fast. 

You can find a sample nginx site config [in our Ansible playbook](https://github.com/engagementlab/el-sdk-playbook/blob/master/roles/nginx/templates/etc/nginx/sites-available/angular.conf.j2).

## SSL using Lets Encrypt/Cerbot

All of our production-level sites use Certbot for SSL certification. Please see [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-let-s-encrypt-with-nginx-server-blocks-on-ubuntu-16-04) for a summary of how this is configured in nginx (Ubuntu).

You can also see how this is configured to use SSL certs in the example nginx site config above, noting that you will also need a directory with root-access called ``/tmp/letsencrypt-auto`` and a location rule for ``"/.well-known/acme-challenge"`` that uses that dir as its root.

_Important_: you may need to setup a cron job such as the one below to keep certificates from expiring. MAKE SURE you test the command to ensure it will run properly. ***Certificate expiration will cause downtime.***

This examples runs our Ansible-managed [renew](https://github.com/engagementlab/el-sdk-playbook/blob/master/roles/nginx/templates/renew.certbot.sh.j2) bash.

```bash
# SSL auto-renew check (every Monday 2:30am)
29 2 * * 1 printf "$(date)\n---------\n\n" >> /var/log/le-renew.log
30 2 * * 1 sh /root/renew.sh
35 2 * * 1 /etc/init.d/nginx reload
```

## pm2

We at the Engagement Lab use [pm2](https://pm2.io/runtime/) to handle our Node production deployments. You don't have to, of course. When you fork the repo, you can always clone/pull and install (not recommended).

If you _do_ want to use pm2, you should read the [docs](https://pm2.io/doc/en/runtime/overview/) to install and set it up on your box. These docs are fairly exhaustive.

### Running the client in production
Run `npm run build` to build the project. The client app artifacts will be stored in the `dist/` directory.  

---
Run `npm i -g http-server`.

---
Run ```NODE_ENV=production pm2 start http-server ---name app-name-client --- dist -p 8080 (or another port)```.

### Running the server in production
From the server directory, run ```NODE_ENV=production pm2 start app ---name app-name-server```.