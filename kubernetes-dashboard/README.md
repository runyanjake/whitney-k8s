# Kubernetes Dashboard

### Setup

Clone the config and start the dashboard:

https://github.com/kubernetes/dashboard

On first startup we have to do authentication 

- Use kubectl port-forward to open the cluster's port 8080 so we can see it over the local network. 

`kubectl port-forward svc/kubernetes-dashboard -n kubernetes-dashboard 8080:443 --address='0.0.0.0'`

> (Other examples will tell you to do `kubectl proxy` which just opens it locally. You'd have to `telnet 127.0.0.1:8001` to see it if you do it that way) 

- Create a Bearer authentication token by following https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md

(See admin-service-account/ for more on generating auth token.)

- Once all this works, permanently set up networking to allow connections to be made from outside the local network.
