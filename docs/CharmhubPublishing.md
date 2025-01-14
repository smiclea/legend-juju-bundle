# FINOS Legend Image Publishing Job on Charmhub

The ``Publish Legend Images`` [action](../.github/workflows/publish_images.yaml) is configured to run periodically, and can be manually triggered by a project collaborator / owner. The action will check any new FINOS Legend releases created in the [FINOS Legend repository](https://github.com/finos/legend), for each new release the action will pull the FINOS Legend Engine, SDLC, and Studio images defined in that release's ``manifest.json`` file, push them as new resource revisions in Charmhub and make a Charm releases using them.

This action requires the ``CHARMHUB_TOKEN`` repository secret (Settings > Secrets > Actions > New repository secret) to be set up in the project in order to push the new charm revisions. The token can be generated by running:

```bash
charmcraft login --export=secrets-legend.auth \
  --charm=finos-legend-engine-k8s --charm=finos-legend-sdlc-k8s \
  --charm=finos-legend-studio-k8s --bundle=finos-legend-bundle \
  --charm=finos-legend-gitlab-integrator-k8s --charm=finos-legend-db-k8s \
  --permission=package-manage --permission=package-view-revisions \
  --ttl=15780000
```

This token will have to be updated periodically since it has a certain time to live set. If the `charmcraft` process fails with the message "Provided credentials are no longer valid for Charmhub. Regenerate them and try again.", it means you have to export the secrets again by logging in to charmcraft using the instructions written above.

By using the following `gh` commands you can skip going through each of the repositories settings to set the secret token:

```bash
gh -R finos/legend-juju-bundle secret set CHARMHUB_TOKEN < secrets-legend.auth
gh -R finos/legend-juju-sdlc-server-operator secret set CHARMHUB_TOKEN < secrets-legend.auth
gh -R finos/legend-juju-engine-server-operator secret set CHARMHUB_TOKEN < secrets-legend.auth
gh -R finos/legend-juju-studio-operator secret set CHARMHUB_TOKEN < secrets-legend.auth
gh -R finos/legend-juju-gitlab-integrator secret set CHARMHUB_TOKEN < secrets-legend.auth
gh -R finos/legend-juju-db-operator secret set CHARMHUB_TOKEN < secrets-legend.auth
```

The ``Publish Legend Images`` action will trigger the ``Create FINOS Legend release branch`` [action](../.github/workflows/create_release.yaml), which will create a new branch in the repository for the new release. This action can also be manually dispatched. But in order for this action chain to work, the ``GH_TOKEN`` repository secret needs to be added. The token can be [generated](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) with the ``repo, workflow`` scopes. The action will also send an email in case of failure, so it needs email related secrets to be set up as well: `EMAIL_SERVER`, `EMAIL_PORT`, `EMAIL_USERNAME`, `EMAIL_PASSWORD`, `EMAIL_TO`, `EMAIL_FROM`.
