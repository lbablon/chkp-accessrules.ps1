
# chkp-accessrules.ps1 - Powershell script to export access rules from a policy

The script uses Checkpoint webservices api to connect to a management server and export all access rules from a specified access layer into a csv file. 

The script as been tested on R81.10 with api version 1.8.1 and above, but it should also works with older versions. No compatibility ith MDS yet. 

## Usage

Mandatory input parameters include management server's ip, user with sufficient permissions, a password and the access layer named after the package policy you want to export rules from. If you do not specify a password and a path when calling the script, it will prompted to you during the execution. Otherwise, the script can be run without any interaction if you set the password and the export path using the parameter switchs.

You can optimize the execution time of the script by changing the limit value of the json body object when querying the rules. This can be done in the following section of the code :

```
#body
$body=@{

"details-level"="standard"
"name"=$AccessLayer
"use-object-dictionary"="false"
"show-hits"="true"
"offset"=$offset
"limit"=100

}
```

Execution time can change depending on your management server's sizing, also i notice that it can greetly improve the cache usage of the api on small sized management server, reducing risk of seeing error 500. Also, the less the limit is, the more precise the progress bar will be.

The final output can be customize following your needs in the code's part where we create the powershell object $allrules : 

```
$allrules=$allrules | % {

    $i=$_
    New-Object -TypeName psobject -Property @{

        'rule-number'=$i.'rule-number'
        'name'=$i.name
        'source'=$i.source.name -join ";"
        'source-negate'=$i.'source-negate'
        'destination'=$i.destination.name -join ";"
        'destination-negate'=$i.'destination-negate'
        'services'=$i.service.name -join ";"
        'action'=$i.action.name
        'track'=$i.track.type.name
        'comments'=$i.comments
        'install-on'=$i.'install-on'.name
        'enabled'=$i.enabled
        'hits'=$i.hits.value
        'hits-percentage'=$i.hits.percentage
        'creation-time'=$i.'meta-info'.'creation-time'.'iso-8601'
        'owner'=$i.'meta-info'.creator
        'last-modify-time'=$i.'meta-info'.'last-modify-time'.'iso-8601'
        'last-modifier'=$i.'meta-info'.'last-modifier'
        'uid'=$i.uid

    }
}
```

Just remove or add the property you need based on [documentation](https://sc1.checkpoint.com/documents/latest/APIs/#cli/show-access-rulebase~v1.8.1%20) and do not forget to adapt the select part of the export command : 

```
$allrules |
    Sort 'rule-number' | 
    Select 'rule-number','name','source','source-negate','destination','destination-negate','services','action','track','comments','install-on','enabled','hits','hits-percentage','creation-time','owner','last-modify-time','last-modifier','uid' |
    Export-Csv -NoTypeInformation -Encoding Default -Delimiter ";" -Path $path
```

## Parameters

- **[-server]**, Checkpoint management server's ip address or fqdn.
- **[-user]**, user with sufficient permissions on the management server.
- **[-password]**, password for the api user.
- **[-accesslayer]**, access layer's name that corresponds to the policy package you want to export rules from.
- **[-path]**, filepath where you want to export the results. This should be a .csv file.

## Examples

```
"./chkp-accessrules.ps1" -Server 192.168.1.50 -user admin -AccessLayer "Standard"
```

Runs the script then asks the user for password then and exports all rules from the access layer named "Standard" then asks the user where to save the results as a csv file.

```
"./chkp-accessrules.ps1" -Server 192.168.1.50 -user admin -password Str0nK! -AccessLayer "Standard" -Path "C:\Temp\rules.csv"
```

Runs the script in non interactive mode and exports the access rules to C:\Temp\rules.csv
