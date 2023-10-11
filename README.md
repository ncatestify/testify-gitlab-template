# testify gitlab template

Einrichtung der Testify QA in eine bestehende Gitlab pipeline


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

Diese wollen wir mit den Testify QA Tests ausstatten.


## Dafür brauchen wir:

 * Eine von Testify bereitgestellte Projekt-Kennung (``TESTIFY_CUSTOMER_NAME``), Umgebungs-Kennung (``TESTIFY_CUSTOMER_ENV``) und einen Access-Token (``REMOTE_GITLAB_TRIGGER_TOKEN``) 

 * Wir erstellen einen Access-Token für das Projekt, das wir selbst einsetzen und auch Testify mitteilen (``PROJECT_API_TOKEN``)

 * Die Kenntnis der zu überprüfenden URL (``BASE_URL``)


## Und ändern damit:

Unsere pipeline triggert nach dem deploy die Testify QA pipeline. Die Testify QA pipeline meldet sich nach Ausführung mit dem Ergebnis zurück (dafür sind die jeweiligen Token erforderlich).

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
  - remote: https://git.nevercodealone.de/testify-infrastructure/testify-gitlab-template/-/raw/master/templates/testify.gitlab-ci.yml
 
qa_external_trigger_job:
  variables:
    # REMOTE_GITLAB_TRIGGER_TOKEN should be set in ci variables as secret, as provided from testify
    # PROJECT_API_TOKEN should be set in ci variables as secret, containing a project token with api access in the current gitlab instance and project
    BASE_URL: "https://nevercodealone.de"
    TESTIFY_CUSTOMER_NAME: "acme-corp"
    TESTIFY_CUSTOMER_ENV: "staging"
```

Nach dem ``deploy_job`` wird dann die Testify pipeline getriggert. Während diese läuft, steht die Pipeline im eigenen Projekt still und der ``qa_external_result_job`` steht auf "manual". Sobald die Testify pipeline durchgelaufen ist, meldet sie sich im ``qa_external_result_job`` mit dem Ergebnis zurück.

Die pipeline kann auf Wunsch nach dem ``qa_external_result_job`` nach belieben erweitert werden (z.B. ``build -> test -> staging deploy → qa → prod deploy`` nur nach Erfolg von qa).


## minimal snippet

Das minimale Snippet wenn alle Variablen über Gitlab CI Variablen gepflegt werden:

```
include:
  - remote: https://git.nevercodealone.de/testify-infrastructure/testify-gitlab-template/-/raw/master/templates/testify.gitlab-ci.yml
```

Achtung: Aktuell ist diese gitlab Instanz (noch) nicht für public repositories ausgelegt, das muss angepasst oder ein anderer Ort gefunden werden.