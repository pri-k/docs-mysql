# Application Security Groups for the p-mysql Service

To allow applications using this service to have access you will need to create
[Application Security Groups](http://docs.pivotal.io/pivotalcf/adminguide/app-sec-groups.html) (ASGs).

<p class="note"><strong>Note</strong>: Without Application Security Groups the service will not be usable.</p>

Application containers that access instances of this service require an outbound network connection to the load
balancer configured for the p-mysql service.

Create a JSON file with the following contents called `p-mysql-security-group.json`:

```
[
  {
      "ports": "3306",
      "protocol": "tcp",
      "destination": "REPLACE WITH THE P-MYSQL LOAD BALANCER IP, RANGE OR CIDR"
  }
]
```


Log in as an administrator and create an ASG called `p-mysql-service`.

    # after logging in as an administrator
    $ cf create-security-group p-mysql-service p-mysql-security-group.json

Bind the new ASG to the `default-running` ASG set so all applications can access the service.

    $ cf bind-running-security-group p-mysql-service

If instead the service should only be available to specific Spaces, bind the
ASG directly to those spaces.

    $ cf bind-security-group p-mysql-service ORGANIZATION_NAME SPACE_NAME
