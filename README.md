# Quelques exemple de requetes REST pour Ansible Tower

Dans cet exemple nous avons un playbook: [tartiflette.yml](tartiflette.yml) qui utilise la variable "ingredients" : 

```
    ingredients:
      vegetables:
        onions: 2
        potatoes: 5
      cheese: "raclette"
```

Ce playbook est utilisé dans un projet et un job template Tower.
La definition du projet et du template peut egalement se faire en REST, mais dans un premier temps on peut le faire avec le GUI ou avec tower-cli: 

```
tower-cli project create --name "Modern Kitchen" --scm-type git --scm-url https://github.com/automaticdavid/modernkitchen.git

tower-cli job_template create --name "Make Tartiflette" --project "Modern Kitchen" --playbook tartiflette.yml --inventory "Default Inventory"
```

On utilise un inventaire par defaut car le playbook ne s'execute que sur localhost. Ce playbook ne fait rien d'autre qu'afficher des messages et des variables, mais cela est suffisant pour démontrer quelques fonctionalités de l'API.

Une fois le template créé on peut lister ses propriétés: 

```
curl --silent --user admin:password "https://tower.example.com/api/v2/job_templates/Make Tartiflette/"
```

On recoit un json de resultat qu'on peut pretty print ou parser, par exemple avec ```jq```: 

```
curl --silent --user admin:password "https://tower.example.com/api/v2/job_templates/Make Tartiflette/" | jq ".name, .summary_fields.last_job"
"Make Tartiflette"
{
  "id": 786,
  "name": "Make Tartiflette",
  "description": "",
  "finished": "2019-02-01T13:06:02.198265Z",
  "status": "successful",
  "failed": false
}
```

On peut declencher l'execution de ce template avec un POST:

```
curl --silent --user admin:password -X POST "https://tower.example.com/api/v2/job_templates/Make Tartiflette/launch/" 
```

On obtient un json qui en partiiculier contient le job ID: 

```
{
  "job": 790,
  "ignored_fields": {},
  "id": 790,
  "type": "job",
  "url": "/api/v2/jobs/790/",
  "related": {
    "created_by": "/api/v2/users/1/",
    "modified_by": "/api/v2/users/1/",
    "labels": "/api/v2/jobs/790/labels/",
    "inventory": "/api/v2/inventories/7/",
    "project": "/api/v2/projects/43/",
    "extra_credentials": "/api/v2/jobs/790/extra_credentials/",
    "credentials": "/api/v2/jobs/790/credentials/",
    "unified_job_template": "/api/v2/job_templates/55/",
    "stdout": "/api/v2/jobs/790/stdout/",
    "notifications": "/api/v2/jobs/790/notifications/",
    "job_host_summaries": "/api/v2/jobs/790/job_host_summaries/",
    "job_events": "/api/v2/jobs/790/job_events/",
    "activity_stream": "/api/v2/jobs/790/activity_stream/",
    "job_template": "/api/v2/job_templates/55/",
    "cancel": "/api/v2/jobs/790/cancel/",
    "create_schedule": "/api/v2/jobs/790/create_schedule/",
    "relaunch": "/api/v2/jobs/790/relaunch/"
  },
  "summary_fields": {
    "job_template": {
      "id": 55,
      "name": "Make Tartiflette",
      "description": ""
    },
    "inventory": {
      "id": 7,
      "name": "Default Inventory",
      "description": "",
      "has_active_failures": false,
      "total_hosts": 3,
      "hosts_with_active_failures": 0,
      "total_groups": 3,
      "groups_with_active_failures": 0,
      "has_inventory_sources": true,
      "total_inventory_sources": 1,
      "inventory_sources_with_failures": 0,
      "organization_id": 1,
      "kind": ""
    },
    "unified_job_template": {
      "id": 55,
      "name": "Make Tartiflette",
      "description": "",
      "unified_job_type": "job"
    },
    "project": {
      "id": 43,
      "name": "Modern Kitchen",
      "description": "",
      "status": "successful",
      "scm_type": "git"
    },
    "created_by": {
      "id": 1,
      "username": "admin",
      "first_name": "",
      "last_name": ""
    },
    "modified_by": {
      "id": 1,
      "username": "admin",
      "first_name": "",
      "last_name": ""
    },
    "user_capabilities": {
      "start": true,
      "delete": true
    },
    "labels": {
      "count": 0,
      "results": []
    },
    "extra_credentials": [],
    "credentials": []
  },
  "created": "2019-02-01T14:29:32.815885Z",
  "modified": "2019-02-01T14:29:32.902898Z",
  "name": "Make Tartiflette",
  "description": "",
  "job_type": "run",
  "inventory": 7,
  "project": 43,
  "playbook": "tartiflette.yml",
  "forks": 0,
  "limit": "",
  "verbosity": 0,
  "extra_vars": "{}",
  "job_tags": "",
  "force_handlers": false,
  "skip_tags": "",
  "start_at_task": "",
  "timeout": 0,
  "use_fact_cache": false,
  "unified_job_template": 55,
  "launch_type": "manual",
  "status": "pending",
  "failed": false,
  "started": null,
  "finished": null,
  "elapsed": 0,
  "job_args": "",
  "job_cwd": "",
  "job_env": {},
  "job_explanation": "",
  "execution_node": "",
  "controller_node": "",
  "result_traceback": "",
  "event_processing_finished": false,
  "job_template": 55,
  "passwords_needed_to_start": [],
  "ask_diff_mode_on_launch": false,
  "ask_variables_on_launch": false,
  "ask_limit_on_launch": false,
  "ask_tags_on_launch": false,
  "ask_skip_tags_on_launch": false,
  "ask_job_type_on_launch": false,
  "ask_verbosity_on_launch": false,
  "ask_inventory_on_launch": false,
  "ask_credential_on_launch": false,
  "allow_simultaneous": false,
  "artifacts": {},
  "scm_revision": "",
  "instance_group": null,
  "diff_mode": false,
  "job_slice_number": 0,
  "job_slice_count": 1,
  "credential": null,
  "vault_credential": null
}
```

On peut vérifier le statut du job:

```
curl --silent --user admin:password "https://tower.example.com/api/v2/jobs/790/" | jq ".name, .id, .status" 
"Make Tartiflette"
790
"successful"
```

On peut extraire la log d'execution du job (differents formats sont disponibles):

```
curl --silent --user admin:password "https://tower.example.com/api/v2/jobs/790/stdout/?format=txt"

PLAY [Let's make a Tartiflette] ************************************************


TASK [Adding Vegetables] *******************************************************
ok: [localhost] => (item={'value': 5, 'key': u'potatoes'}) => {
    "msg": "Adding 5 potatoes"
}
ok: [localhost] => (item={'value': 2, 'key': u'onions'}) => {
    "msg": "Adding 2 onions"
}

TASK [Adding Cheese] ***********************************************************

ok: [localhost] => {
    "msg": "Adding raclette"
}

TASK [Make Tartiflette] ********************************************************
ok: [localhost] => {
    "ingredients": {
        "cheese": "raclette", 
        "vegetables": {
            "onions": 2, 
            "potatoes": 5
        }
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=0    unreachable=0    failed=0   
```

On peut surcharger les variables d'execution du template en passant un json contenant des extra_vars. Pour cela il faut s'assurer que la case "prompt on launch" du job template est cochée pour les extra_vars. On peut le faire dans l'interface graphique ou avec la commande tower-cli suivante: 

```
tower-cli job_template modify --name "Make Tartiflette"  --ask-variables-on-launch true
== ================ ========= ======= =============== 
id       name       inventory project    playbook     
== ================ ========= ======= =============== 
55 Make Tartiflette         7      43 tartiflette.yml
== ================ ========= ======= =============== 
```

On peut alors passer un json au POST REST permettant de surcharger les variables du playbook:

```
cat ingredients.json 
"ingredients":
    {
        "vegetables":
            {
                "onions": 5,
                "potatoes": 15
            },
        "cheese": "raclette"
    }

curl --silent --user admin:password -X POST -H "Content-Type: application/json" -d @ingredients.rest.json "https://tower.example.com/api/v2/job_templates/Make Tartiflette/launch/"

```

En regardant le statut du job on peut voir que les extravars ont été prises en compte:

```
curl --silent --user admin:password "https://tower.example.com/api/v2/jobs/800/" | jq -r ".extra_vars" | jq
{
  "ingredients": {
    "vegetables": {
      "onions": 5,
      "potatoes": 15
    },
    "cheese": "raclette"
  }
}
```




