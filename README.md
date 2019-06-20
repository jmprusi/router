# Custom openshift router

**PoC for a Custom Openshift-router with 3scale authorization capabilities** 

**Don't use in production**

This is a modified version of the openshift-router to check if it makes sense to add 3scale authorization capabilities to 
the openshift default ingress.


## Configuration

Replace your router image:

```bash
oc patch dc router -p '{"spec":{"template":{"spec":{"containers":[{"name":"router","image":"quay.io/jmprusi/openshift-custom-router:latest"}]}}}}'
```

Add the 3scale-haproxy container:

```yaml
      - image: quay.io/jmprusi/3scale-haproxy:latest
        imagePullPolicy: Always
        name: 3scale-haproxy
        ports:
        - containerPort: 12345
          hostPort: 12345
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
```

Enable the 3scale authentication with the ENV var:

```bash
oc set env dc/router ROUTER_ENABLE_THREESCALE_SPOA=true -n default -c router
```

## Protecting a Route

To protect a Route, you need to add those annotations: 

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    haproxy.router.openshift.io/3scale-secured-api: "True"
    haproxy.router.openshift.io/3scale-serviceid: "111111111"
    haproxy.router.openshift.io/3scale-systemurl: https://myaccount-admin.3scale.net:443/
    haproxy.router.openshift.io/3scale-test: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

The 3scale account should be configured with the "on-premises apicast" option, and the configuration should be published to the production environment of your 3scale account.

Works with 3scale on-premises or SaaS.

 