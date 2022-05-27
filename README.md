[![ci](https://github.com/paperless-ngx/paperless-ngx/workflows/ci/badge.svg)](https://github.com/paperless-ngx/paperless-ngx/actions)
[![Crowdin](https://badges.crowdin.net/paperless-ngx/localized.svg)](https://crowdin.com/project/paperless-ngx)
[![Documentation Status](https://readthedocs.org/projects/paperless-ngx/badge/?version=latest)](https://paperless-ngx.readthedocs.io/en/latest/?badge=latest)
[![Coverage Status](https://coveralls.io/repos/github/paperless-ngx/paperless-ngx/badge.svg?branch=master)](https://coveralls.io/github/paperless-ngx/paperless-ngx?branch=master)
[![Chat on Matrix](https://matrix.to/img/matrix-badge.svg)](https://matrix.to/#/#paperless:adnidor.de)

<p align="center">
<img src="https://github.com/paperless-ngx/paperless-ngx/raw/main/resources/logo/web/png/Black%20logo%20-%20no%20background.png#gh-light-mode-only" width="50%" />
<img src="https://github.com/paperless-ngx/paperless-ngx/raw/main/resources/logo/web/png/White%20logo%20-%20no%20background.png#gh-dark-mode-only" width="50%" />
</p>

<!-- omit in toc -->

# Paperless-ngx

Paperless-ngx is a document management system that transforms your physical documents into a searchable online archive so you can keep, well, _less paper_.

Paperless-ngx forked from [paperless-ng](https://github.com/jonaswinkler/paperless-ng) to continue the great work and distribute responsibility of supporting and advancing the project among a team of people. [Consider joining us!](#community-support) Discussion of this transition can be found in issues
[#1599](https://github.com/jonaswinkler/paperless-ng/issues/1599) and [#1632](https://github.com/jonaswinkler/paperless-ng/issues/1632).

# Paperless-ngx on Kubernetes
I don't care...take me to [TLDR](#TLDR)

I ran paperless-ngx on Docker for a while and moved to installation to Kubernetes.  I did this for a couple of reasons
- Learn Kubernetes better
- Help out people who wanted to learn Kubernetes better
- See how a document management sytem would scale and work for me

This is repo of the manifiests that I used to put paperless into my microk8s cluster.  There are a few caveats here:
- If you run multiple nodes in your cluster your PVC configs will beed to refelct that, OpenEBS is a good option
- I put the consumption directory on and NFS share - why you ask?  Simple
    - If you have a scanner you can scan directly to a share on a NAS, File server or whatever you want
    - Usually this is a share folder somewhere, created a PVC for this seemed like a bad idea a share was best
    - You can be grandualr with permissons on the share so make sure you grant paperless the access needed
    - You can also have a "inotify" process running to pass stuff to this directory from another share if you want.

The possibilities are endless really

I also offloaded OCR and document convertion to Tika and Gotenberg respectively.  The OCR deployment and service manifests show the servicies neded.

I also put no NGINX ingress on this installation as I didn't want it, I wanted the port.  In my setup I have an external LB/Controller that handles access-lists and certificates. You can easly change the service deployments for the webserver to have ingress if you which then create the ingress manifest which I may include later.

You'll also notice an AV manifest.  I was working on a solution to scan uploads with ClamAV but it's not there yet so you can safely remove them if you want or keep thema and see if you can get Clam to scan the consume directory.

# Installation
Have a working K8S installaton somwhere.  Microk8s, Minikube...doesn't matter.  Download/pull the manifest and edit paperless-config.yaml to your liking.  You'll notice those are the envrionment values for paperless itself so you can easily add/remove what you want based on the paperless documentation [here](https://paperless-ng.readthedocs.io/en/latest/configuration.html).  Please note the strings for the OCR section, as they point to the OCR Service depolyment.  This DNS internal to the cluster will resolve the service name not the names of the containers so make sure you don't change that or try to resolve the container names as you would with docker.  If you have those services running somewhere else like different tenant or not in K8S you'll have to use the IP address. Be careful upgrading the TIKA version past the 1 series branch is it does break stuff.  I might try it later with Tensor-Flow but who knows.

All deployment should pull their configs from configmaps.

If you want to run this in production create a secret for the PosgresSQL database login info and change the env values deployment manifest to reflect that.  Using secrets is easy and you can do that by looking [here](https://kubernetes.io/docs/concepts/configuration/secret/).  Then change the env vaules to: 
    envFrom:
          - secretRef:
              name: your-paperless-db-secret
Or whatever you want to call your secret.  

That's it.  

# TLDR
1. Go to CLI
    ```kubectl create namespace paperless```
2. Edit paperless-config.yaml to your liking 
3. Back to CLI
    ```kubectl -n paperless apply -f .```
4. Profit