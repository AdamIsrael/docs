who: ericsnow, urulama, natefinch, fabrice



charm push --resource is the equivalent of charm push && juju push-resource

1. Declare resources in metadata.yaml
2. Push charm: `charm push . cs:~aisrael/trusty/resources--example`
` --resource word=` is optional, can be repeated
2. Push resource (`charm attach` vs `juju attach`)

`juju attach` uploads it to a controller. `charm attach` uploads it to the charm
store for public consumption.

`charm attach` is currently broken
```
TODO: file bug to make this error more friendly
$ charm publish cs:~aisrael/trusty/resources-example-0 --channel=development
ERROR cannot publish charm or bundle: bad request: charm published with incorrect resources: resources are missing from publish request: words
```
2. Publish charm


Charm deploys
- hook calls resource-get to get the local path to the resource

A resource can be uploaded directly to the controller via push-resource, which will trigger the `upgrade-charm` hook.

From the charm:

    resource-get


push-resource will trigger charm-upgrade (for local uploads?)

<natefinch> marcoceppi: note that push-resource will fire upgrade-charm, so you can just put your "when a new resource is available" code there... though if you deploy --resource foo=./bar.zip then it'll be available even before the install hook
