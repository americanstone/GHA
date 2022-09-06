# create deploy matrix for common application deployment

_DRY. no more duplicated nonprod-deploy or prod-deploy actions. one action rule them all_

```yaml
   # create a matrix from this action
   deploy-matrix-prep:
     if: ${{ github.ref == 'refs/heads/master' || github.ref == 'refs/heads/main' || github.ref == 'refs/heads/env/nonprod' || startsWith(github.ref, 'refs/heads/integration') }}
     runs-on: CAI-Enterprise
     steps:
       - name: matrix
         id: matrix
         uses: DDC-WebPlatform/deploy-matrix-generator@v1.0.9
     outputs:
       matrix: ${{ steps.matrix.outputs.deploy-matrix }}

   # in your project, using the matrix to generate the deployment jobs dynamically 
   deploy:
     needs: [build, deploy-matrix-prep]
     runs-on: CAI-Enterprise
     strategy:
       fail-fast: true
       matrix: ${{fromJSON(needs.deploy-matrix-prep.outputs.matrix) }}

      .....
     - name: Blue Green Service Deploy
       id: deploy
       uses: DDC-WebPlatform/ecs-blue-green-deployment@v1.3
       timeout-minutes: 15
       with:
         region: ${{ matrix.region }}
         environment: ${{ matrix.env }}
         aws-account: ${{  matrix.account }}
```


_for master and main branches, the matrix object looks like following_
```
{env: prod, region: us-east-1, account: 941071384133}
{env: prod, region: us-west-2, account: 941071384133}
```

_for env/nonprod and integration/* branches, the matrix object looks like following_
```
{env: nonprod, region: us-east-1, account: 247721768464}
{env: nonprod, region: us-west-2, account: 247721768464}
```

_for feature/* branch, no matrix will be generated_
