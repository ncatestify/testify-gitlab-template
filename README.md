# TESTIFY gitlab template

Einrichtung der TESTIFY QA in eine bestehende Gitlab Pipeline

## Ausgehend von:

In diesem Beispiel nehmen wir uns eine Pipeline mit den Stages:

``build → test → deploy``

```
# example .gitlab-ci.yml
---
stages:
  - build
  - test
  - deploy
 
build_job:
  - [..]
 
test_job:
  - [..]
 
deploy_job:
  - [..]
```

Diese wollen wir mit den TESTIFY QA Tests ausstatten.


## Dafür brauchen wir:

 * Eine von TESTIFY bereitgestellte Projekt-Kennung (``TESTIFY_CUSTOMER_NAME``), Umgebungs-Kennung (``TESTIFY_CUSTOMER_ENV``) und einen Access-Token (``REMOTE_GITLAB_TRIGGER_TOKEN``) 

 * Wir erstellen einen Access-Token für das Projekt, das wir selbst einsetzen und auch TESTIFY mitteilen (``PROJECT_API_TOKEN``)

 * Die Kenntnis der zu überprüfenden URL (``BASE_URL``)


## Und ändern damit:

Unsere Pipeline triggert nach dem deploy die TESTIFY QA Pipeline. Die TESTIFY QA Pipeline meldet sich nach Ausführung mit dem Ergebnis zurück (dafür sind die jeweiligen Token erforderlich).

``build → test → deploy → qa → result``

```
# example .gitlab-ci.yml
---
stages:
  - build
  - test
  - deploy
  - qa-external-trigger
  - qa-external-result
 
build_job:
  - [..]
 
test_job:
  - [..]
 
deploy_job:
  - [..]
 
include:
  - remote: https://github.com/ncatestify/testify-gitlab-template/blob/main/templates/testify.gitlab-ci.yml
 
qa_external_trigger_job:
  variables:
    # REMOTE_GITLAB_TRIGGER_TOKEN should be set in ci variables as secret, as provided from TESTIFY
    # PROJECT_API_TOKEN should be set in ci variables as secret, containing a project token with api access in the current gitlab instance and project
    BASE_URL: "https://nevercodealone.de"
    TESTIFY_CUSTOMER_NAME: "acme-corp"
    TESTIFY_CUSTOMER_ENV: "staging"
```

Nach dem ``deploy_job`` wird dann die TESTIFY Pipeline getriggert. Während diese läuft, steht die Pipeline im eigenen Projekt still und der ``qa_external_result_job`` steht auf "manual". Sobald die TESTIFY Pipeline durchgelaufen ist, meldet sie sich im ``qa_external_result_job`` mit dem Ergebnis zurück.

Die Pipeline kann auf Wunsch nach dem ``qa_external_result_job`` nach belieben erweitert werden (z.B. ``build -> test -> staging deploy → qa → prod deploy`` nur nach Erfolg von qa).

## minimal snippet

Das minimale Snippet wenn alle Variablen über Gitlab CI Variablen gepflegt werden:

```
include:
  - remote: https://git.nevercodealone.de/TESTIFY-infrastructure/TESTIFY-gitlab-template/-/raw/master/templates/TESTIFY.gitlab-ci.yml
```

Erwartete Variablen:

 * ``TESTIFY_CUSTOMER_NAME``
 
 * ``TESTIFY_CUSTOMER_ENV``
 
 * ``BASE_URL``
 
 * ``REMOTE_GITLAB_TRIGGER_TOKEN``

 * ``PROJECT_API_TOKEN``

### Leute hinter dem Projekt
Das Projekt ist eine Kooperation zwischen Karsten Deubert von (Deubert IT)[https://deubert.it/] und Roland Golla von [TESTIFY.TEAM](https://testify.team/de). 

Die Idee hinter dem Projekt ist hier dokumentiert.
https://testify.team/de/produkte/tt-pipeline-snippet

