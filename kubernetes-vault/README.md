# Выполнено ДЗ № 11

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:

 - Инсталляция hashicorp vault HA в k8s
Работа выполняется в gcp.
Клонируем репозиторий consul.
```
git clone https://github.com/hashicorp/consul-helm.git
```
 - Включаем для cunsul связь по https
```
kubectl create secret generic consul-gossip-encryption-key  --from-literal=key=$(consul keygen)
```
Меняем параметры в consul-helm values.yaml
```
global
  gossipEncryption:
    secretName: consul-gossip-encryption-key
    secretKey: key
```
Обновляем consul-helm
```
helm upgrade --install consul consul-helm -f consul-helm/values.yaml
```
Клонируем репозиторий vault и меняем values.yaml, разворачиваем vault
```
git clone https://github.com/hashicorp/vault-helm.git

...
  standalone:
    enabled: false
...
  ha:
    enabled: true
    replicas: 3
...
ui:
  enabled: true
  serviceType: "ClusterIP"
...

helm install vault vault-helm
```
Проверяем статус и логи vault pods
```
helm status vault

NAME: vault
LAST DEPLOYED: Mon Apr  6 20:59:34 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!
...


kubectl logs vault-0

==> Vault server configuration:
             Api Address: http://10.244.4.6:8200
                     Cgo: disabled
         Cluster Address: https://10.244.4.6:8201
              Listener 1: tcp (addr: "[::]:8200", cluster address: "[::]:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: true, enabled: false
           Recovery Mode: false
                 Storage: consul (HA available)
                 Version: Vault v1.3.3
2020-04-06T17:59:46.338Z [INFO]  proxy environment: http_proxy= https_proxy= no_proxy=
2020-04-06T17:59:46.338Z [WARN]  storage.consul: appending trailing forward slash to path
==> Vault server started! Log data will stream in below:

2020-04-06T17:59:54.106Z [INFO]  core: seal configuration missing, not initialized
2020-04-06T17:59:57.064Z [INFO]  core: seal configuration missing, not initialized
2020-04-06T18:00:00.048Z [INFO]  core: seal configuration missing, not initialized
...
```
 - Инициализируем vault
```
kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1

Unseal Key 1: POgsrEHmDSbz4DMnN8O0NVLIxoNPn5pvpCuuf0irZxA=
Initial Root Token: s.kyvO4esNenP159OmWICLzXBf
```
 - Проверим состояние vault'а
```
kubectl logs vault-0
...
2020-04-06T18:12:27.006Z [INFO]  core: seal configuration missing, not initialized
2020-04-06T18:12:27.830Z [ERROR] core: no seal config found, can't determine if legacy or new-style shamir
2020-04-06T18:12:27.835Z [INFO]  core: security barrier not initialized
2020-04-06T18:12:27.878Z [INFO]  core: security barrier initialized: stored=1 shares=1 threshold=1
2020-04-06T18:12:27.952Z [INFO]  core: post-unseal setup starting
2020-04-06T18:12:27.988Z [INFO]  core: loaded wrapping token key
2020-04-06T18:12:27.988Z [INFO]  core: successfully setup plugin catalog: plugin-directory=
2020-04-06T18:12:27.993Z [INFO]  core: no mounts; adding default mount table
2020-04-06T18:12:28.016Z [INFO]  core: successfully mounted backend: type=cubbyhole path=cubbyhole/
2020-04-06T18:12:28.016Z [INFO]  core: successfully mounted backend: type=system path=sys/
2020-04-06T18:12:28.016Z [INFO]  core: successfully mounted backend: type=identity path=identity/
2020-04-06T18:12:28.141Z [INFO]  core: successfully enabled credential backend: type=token path=token/
2020-04-06T18:12:28.142Z [INFO]  core: restoring leases
2020-04-06T18:12:28.142Z [INFO]  rollback: starting rollback manager
2020-04-06T18:12:28.146Z [INFO]  expiration: lease restore complete
2020-04-06T18:12:28.186Z [INFO]  identity: entities restored
2020-04-06T18:12:28.187Z [INFO]  identity: groups restored
2020-04-06T18:12:28.195Z [INFO]  core: post-unseal setup complete
2020-04-06T18:12:28.255Z [INFO]  core: root token generated
2020-04-06T18:12:28.255Z [INFO]  core: pre-seal teardown starting
2020-04-06T18:12:28.255Z [INFO]  rollback: stopping rollback manager
2020-04-06T18:12:28.255Z [INFO]  core: pre-seal teardown complete


kubectl exec -it vault-0 -- vault status

Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       1
Threshold          1
Unseal Progress    0/1
Unseal Nonce       n/a
Version            1.3.3
HA Enabled         true
```
 - Распечатаем vault
Распечатываем каждый под, вводим Unseal Key для каждого
```
kubectl exec -it vault-0 -- vault operator unseal
kubectl exec -it vault-1 -- vault operator unseal
kubectl exec -it vault-2 -- vault operator unseal
```
Статус подов
```
kubectl exec -it vault-0 -- vault status

Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.3.3
Cluster Name    vault-cluster-8150de57
Cluster ID      45b039b9-df95-8baf-fb87-fea044e31d6f
HA Enabled      true
HA Cluster      https://10.244.4.6:8201
HA Mode         active


kubectl exec -it vault-1 -- vault status

Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.3.3
Cluster Name           vault-cluster-8150de57
Cluster ID             45b039b9-df95-8baf-fb87-fea044e31d6f
HA Enabled             true
HA Cluster             https://10.244.4.6:8201
HA Mode                standby
Active Node Address    http://10.244.4.6:8200


kubectl exec -it vault-2 -- vault status

Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.3.3
Cluster Name           vault-cluster-8150de57
Cluster ID             45b039b9-df95-8baf-fb87-fea044e31d6f
HA Enabled             true
HA Cluster             https://10.244.4.6:8201
HA Mode                standby
Active Node Address    http://10.244.4.6:8200
```
 - Посмотрим список доступных авторизаций
При просмотре получаем ошибку
```
kubectl exec -it vault-0 -- vault auth list

Error listing enabled authentications: Error making API request.
URL: GET http://127.0.0.1:8200/v1/sys/auth
Code: 400. Errors:
* missing client token
```
 - Залогинимся в vault 
```
kubectl exec -it vault-0 --  vault login

Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
Key                  Value
---                  -----
token                s.kyvO4esNenP159OmWICLzXBf
token_accessor       cCl3TAcit3DTHTyJbAuDPWXz
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```
Запрашиваем список авторизаций
```
kubectl exec -it vault-0 -- vault auth list

Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_e129bcd7    token based credentials
```
 - Заведем секреты
```
kubectl exec -it vault-0 -- vault secrets enable --path=otus kv


kubectl exec -it vault-0 -- vault secrets list --detailed

Path          Plugin       Accessor              Default TTL    Max TTL    Force No Cache    Replication    Seal Wrap    External Entropy Access    Options    Description                                                UUID
----          ------       --------              -----------    -------    --------------    -----------    ---------    -----------------------    -------    -----------                                                ----
cubbyhole/    cubbyhole    cubbyhole_f7cb3837    n/a            n/a        false             local          false        false                      map[]      per-token private secret storage                           2ba42f97-f3d1-9030-057d-db8533bd3e54
identity/     identity     identity_9bc767e7     system         system     false             replicated     false        false                      map[]      identity store                                             88ac212d-7018-8daa-d335-a738b1c1c4a0
otus/         kv           kv_05937159           system         system     false             replicated     false        false                      map[]      n/a                                                        668742a2-e1f1-8829-6ee7-300d3f9ea8aa
sys/          system       system_85958804       n/a            n/a        false             replicated     false        false                      map[]      system endpoints used for control, policy and debugging    558d2c80-72f2-0e8b-de5b-c53e7534e044


kubectl exec -it vault-0 -- vault kv put otus/otus-ro/config username='otus' password='asajkjkahs'


kubectl exec -it vault-0 -- vault kv put otus/otus-rw/config username='otus' password='asajkjkahs'


kubectl exec -it vault-0 -- vault read otus/otus-ro/config

Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus


kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config

====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```
 - Включим авторизацию через k8s
```
kubectl exec -it vault-0 -- vault auth enable kubernetes


kubectl exec -it vault-0 --  vault auth list

Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_5dc32102    n/a
token/         token         auth_token_e129bcd7         token based credentials
```
 - Создадим yaml для ClusterRoleBinding
```
vault-auth-service-account.yaml

apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: role-tokenreview-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: vault-auth
    namespace: default
```
 - Создадим Service Account vault-auth и применим ClusterRoleBinding
```
kubectl create serviceaccount vault-auth

kubectl apply -f vault-auth-service-account.yaml
```
 - Подготовим переменные для записи в конфиг кубер авторизации
```
export VAULT_SA_NAME=$(kubectl get sa vault-auth -o jsonpath="{.secrets[*]['name']}")
export SA_JWT_TOKEN=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data.token}" | base64 --decode; echo)
export SA_CA_CRT=$(kubectl get secret $VAULT_SA_NAME -o jsonpath="{.data['ca\.crt']}" | base64 --decode; echo)
export K8S_HOST=$(kubectl cluster-info | grep 'Kubernetes master' | awk '/https/ {print$NF}' | sed 's/\x1b\[[0-9;]*m//g')
```
Строка - sed ’s/\x1b\[[0-9;]*m//g’ - убирает цвет вывода
 - Запишем конфиг в vault
```
kubectl exec -it vault-0 -- vault write auth/kubernetes/config \
  token_reviewer_jwt="$SA_JWT_TOKEN" \
  kubernetes_host="$K8S_HOST" \
  kubernetes_ca_cert="$SA_CA_CRT"

Success! Data written to: auth/kubernetes/config
```
 - Создадим файл политики
```
echo 'path "otus/otus-ro/*" {
capabilities = ["read", "list"]
}

path "otus/otus-rw/*" {
capabilities = ["read", "create", "list"]
}
' | tee otus-policy.hcl
```
 - создадим политку и роль в vault
```
kubectl cp otus-policy.hcl vault-0:/home/vault/

kubectl exec -it vault-0 -- vault policy write otus-policy /home/vault/otus-policy.hcl

kubectl exec -it vault-0 -- vault write auth/kubernetes/role/otus \
	bound_service_account_names=vault-auth \
	bound_service_account_namespaces=default policies=otus-policy ttl=24h
```
 - Проверим как работает авторизация
Создадим под с привязанным сервис аккоунтом
```
kubectl run --generator=run-pod/v1 tmp --rm -i --tty --serviceaccount=vault-auth --image alpine:3.7 sh
apk add curl jq
```
Получаем клиентский токен
```
VAULT_ADDR=http://vault:8200

KUBE_TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token) 
```
output
```
curl -sb --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

{
  "request_id": "2b0e4d37-b992-07b6-3e74-644455b8a76a",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 0,
  "data": null,
  "wrap_info": null,
  "warnings": null,
  "auth": {
    "client_token": "s.bmFCyoWeBptuaIkMjknK31fO",
    "accessor": "IPxyA47VlMBCF8nWA28GJrRr",
    "policies": [
      "default",
      "otus-policy"
    ],
    "token_policies": [
      "default",
      "otus-policy"
    ],
    "metadata": {
      "role": "otus",
      "service_account_name": "vault-auth",
      "service_account_namespace": "default",
      "service_account_secret_name": "vault-auth-token-vwjps",
      "service_account_uid": "73716782-d5c6-467c-8d79-1ed0f2df34e6"
    },
    "lease_duration": 86400,
    "renewable": true,
    "entity_id": "3922d7b5-38ff-7d3e-4729-14cbffd61600",
    "token_type": "service",
    "orphan": true
  }
}
```
!Внимание!
В kind в кластере из 3 нод curl запрос не работет.
```
curl -sb --request POST --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq

{
  "errors": [
    "permission denied"
  ]
}
```
Клиентский токен
```
TOKEN=$(curl -k -s --request POST  --data '{"jwt": "'$KUBE_TOKEN'", "role": "otus"}' $VAULT_ADDR/v1/auth/kubernetes/login | jq '.auth.client_token' | awk -F\" '{print $2}')

echo $TOKEN

s.PbkEyjlRudS4CSPuPKmaJ6Dd
```
 - Прочитаем записанные ранее секреты и попробуем их обновить
Используем полученный клиенский токен и проверяем чтение и запись
Чтение
```
curl --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-ro/config | jq

{
  "request_id": "78108c61-aee6-a45f-e152-a7cae19da2a9",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 2764800,
  "data": {
    "password": "asajkjkahs",
    "username": "otus"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}

curl --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config | jq

{
  "request_id": "a0f2bdcc-b465-f3e7-aa16-c75aac4ee521",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 2764800,
  "data": {
    "password": "asajkjkahs",
    "username": "otus"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```
Запись
```
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-ro/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}

curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config
{"errors":["1 error occurred:\n\t* permission denied\n\n"]}

curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config1

curl --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config1 | jq
{
  "request_id": "c393ecbd-9d75-c298-1013-8cb7fbd5cbf8",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 2764800,
  "data": {
    "bar": "baz"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```
 - Разберемся с ошибками при записи
Запись в otus-rw/config не получилась, т.к. конфигурация ранее заданной политике это не позваляет.
Исправляем
```
echo 'path "otus/otus-ro/*" {
capabilities = ["read", "list"]
}

path "otus/otus-rw/*" {
capabilities = ["read", "create", "list", "update"]
}
' | tee otus-policy-update.hcl

kubectl cp otus-policy-update.hcl vault-0:/home/vault/

kubectl exec -it vault-0 -- vault policy write otus-policy /home/vault/otus-policy-update.hcl
```
Проверяем
```
curl --request POST --data '{"bar": "baz"}' --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config

curl --header "X-Vault-Token:s.PbkEyjlRudS4CSPuPKmaJ6Dd" $VAULT_ADDR/v1/otus/otus-rw/config | jq
{
  "request_id": "9ef5ce96-43a1-722f-9d19-8bd5eaea7736",
  "lease_id": "",
  "renewable": false,
  "lease_duration": 2764800,
  "data": {
    "bar": "baz"
  },
  "wrap_info": null,
  "warnings": null,
  "auth": null
}
```
 - Использование авторизации через кубер
Берем репозиторий с примерами
```
git clone https://github.com/hashicorp/vault-guides.git
```
Исправляем example-k8s-spec.yml
```
- name: VAULT_ADDR
  value: http://vault:8200
```
Исправляем configs-k8s/vault-agent-config.hcl
```
            role = "otus"
```
Исправляем configs-k8s/consul-template-config.hcl
```
  {{- with secret "otus/otus-ro/config" }}
```
Запускаем пример
```
kubectl create configmap example-vault-agent-config --from-file=./vault-guides/identity/vault-agent-k8s-demo/configs-k8s/

kubectl get configmaps example-vault-agent-config -o yaml

apiVersion: v1
data:
  consul-template-config.hcl: |
    vault {
      renew_token = false
      vault_agent_token_file = "/home/vault/.vault-token"
      retry {
        backoff = "1s"
      }
    }

    template {
      destination = "/etc/secrets/index.html"
      contents = <<EOH
      <html>
      <body>
      <p>Some secrets:</p>
      {{- with secret "otus/otus-ro/config" }}
      <ul>
      <li><pre>username: {{ .Data.username }}</pre></li>
      <li><pre>password: {{ .Data.password }}</pre></li>
      </ul>
      {{ end }}
      </body>
      </html>
      EOH
    }
  vault-agent-config.hcl: |
    # Uncomment this to have Agent run once (e.g. when running as an initContainer)
    exit_after_auth = true
    pid_file = "/home/vault/pidfile"

    auto_auth {
        method "kubernetes" {
            mount_path = "auth/kubernetes"
            config = {
                role = "otus"
            }
        }

        sink "file" {
            config = {
                path = "/home/vault/.vault-token"
            }
        }
    }
kind: ConfigMap
metadata:
  creationTimestamp: "2020-04-19T19:29:54Z"
  name: example-vault-agent-config
  namespace: default
  resourceVersion: "3047563"
  selfLink: /api/v1/namespaces/default/configmaps/example-vault-agent-config
  uid: c5f2b137-a148-4142-9dd1-0d7f2d48c6e0
```
Создаем vault-agent-example Pod
```
kubectl apply -f ./vault-guides/identity/vault-agent-k8s-demo/example-k8s-spec.yml --record
```
Пробрасываем порты
```
kubectl port-forward vault-agent-example 8080:80
```
Смотрим на 127.0.0.1:8080
```
curl -s 127.0.0.1:8080

  <html>
  <body>
  <p>Some secrets:</p>
  <ul>
  <li><pre>username: otus</pre></li>
  <li><pre>password: asajkjkahs</pre></li>
  </ul>
  
  </body>
  </html>
```
 - создадим CA на базе vault
```
kubectl exec -it vault-0 -- vault secrets enable pki

kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki

kubectl exec -it vault-0 -- vault write -field=certificate pki/root/generate/internal \
	common_name="exmaple.ru"  ttl=87600h > CA_cert.crt
```
output
```
-----BEGIN CERTIFICATE-----
MIIDMjCCAhqgAwIBAgIUN94kO4Cg0Vq9OVeuL8PH3fIElK8wDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDA0MTkyMTAwMjlaFw0zMDA0
MTcyMTAwNTlaMBUxEzARBgNVBAMTCmV4bWFwbGUucnUwggEiMA0GCSqGSIb3DQEB
AQUAA4IBDwAwggEKAoIBAQCqnuz9Vzm5J6L3Sa6gvVf6nYChun52ezYme/S/VMl5
cDOZxWzBaZNd/isAYAliWZkrn0xE8Ym17o7xP8/wJFNglWcxhODKTEhOdETxF3Lv
d7mF5TAwGIDKCQkvZJ2C5wNda+ICbtvLVG4bZVdlQDopcXP7KzG9gAbz+W9httTQ
gWnD8ICCT4pOOdG2xPRb0z5ism6/obvNESH950INafxptXQ08HuKCSwPGyvEe6Sx
0ZXJdFJmesGYD5nBdGVyq+Nj+CTK1u0qdkAKo2jInJDk30m/45NzVaxAdbZNYlgb
erukrjVYE2a6I1i93pchbemUhZRx6RYwC6I4yZ5PsPxxAgMBAAGjejB4MA4GA1Ud
DwEB/wQEAwIBBjAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQ9Q0ulwutRIrs0
oFtKWgDhv3jhmDAfBgNVHSMEGDAWgBQ9Q0ulwutRIrs0oFtKWgDhv3jhmDAVBgNV
HREEDjAMggpleG1hcGxlLnJ1MA0GCSqGSIb3DQEBCwUAA4IBAQBNFWmBRZpaIGe4
z+tURlINFxzrelZVwsezVzk/0oPMBHSrYNjmJPqhB2wS042N8krtRUxiIZaqrxEC
J1uIWXnUA3pR4+mlRTN5b9zgD4qSnabw3gnbPwTvBEDgdu1YO9BoqF96f19YSoXx
dCzmGoVV9dUFEPElAXaVfEBRjcDTswF8z6TNhl+TsWINLtP/5OLLCl9gtxMxAelo
nymFQkwTOYNeI5ttTbl7u2R3znA45NDLINimNk+spkU4c1hvWrSaO3jKMpGOpcRh
MIvWBgjQXbPvLlmV4LdOANYQo7bKVdUEGRvgshRHee/Z1DvDNjPGBZkBmkGZkMdQ
JscxQetB
-----END CERTIFICATE-----
```
 - пропишем URL для CA и отозванных сертификатов
```
kubectl exec -it vault-0 -- vault write pki/config/urls \
	issuing_certificates="http://vault:8200/v1/pki/ca" \
	crl_distribution_points="http://vault:8200/v1/pki/crl"
```
 - создадим промежуточный сертификат
```
kubectl exec -it vault-0 -- vault secrets enable --path=pki_int pki

kubectl exec -it vault-0 -- vault secrets tune -max-lease-ttl=87600h pki_int

kubectl exec -it vault-0 -- vault write -format=json pki_int/intermediate/generate/internal \
	common_name="example.ru Intermediate Authority" | jq -r '.data.csr' > pki_intermediate.csr
```
output
```
-----BEGIN CERTIFICATE REQUEST-----
MIICcTCCAVkCAQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUg
QXV0aG9yaXR5MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtmOE6VJj
LyFCt9I/L1IwOGg9XuNXRg2J0JmZOVdk0kiJ4oTW0I1KJQepya29o83XmoO1/8uq
J4xwT3l5qI9RRTrNym8RYYFmQnaVobZfy57z4phUtxzQBBNkdtS+GT8NdDT7A2kM
vwmF+v64ysJrdxg0RZYMd4jVxribZYIVei1goxorbOJYTwV5GY63ZgdprbM9Nb3S
mb3qDqb0AOXUYqeX5PvVLgL+VRyeVKjgSRyHupHZY8/GJ5/bfupG1cgDdR6CuiEI
w8hZCuwdapU/YYJu07cKFX+M4vpqY1BIrxgjxRECgNACmHP6cm9iwyjnf1NewOyS
37VrdDVQ+IJdhQIDAQABoAAwDQYJKoZIhvcNAQELBQADggEBAB7/bnFFCoXV3XIW
+ckFy01NhBLZGJmRnN8Si75yoB07q+WCE85Ny9HWIPLTeAmHbBaRa+agouUAS3+k
bvJVAo8GYSInCVBqQ1fJrGNfBveWPqrBTFXAhwOkRsVcmgz5gF+Jde/jvXaFVWx8
5xSOeh+cf5SjgXw2JFoEpAceZRZjEzk/+C88t3x+Lw4wlzUuKazIaTQkI2o8QwWO
i2HPCLsiRge7p4QQLXr5ExjQei2YsgkHrLm0WSJNNcNMJb2rJn1KAPrKPv84WzMF
ojZypuGMi7FKtYAw+Mq7Sg8DnEeU2z9yq2cCd+/ANCxy0oTZH+WxO9eHaRVq+c6f
4JA8vLY=
-----END CERTIFICATE REQUEST-----
```
 - пропишем промежуточный сертификат в vault
```
kubectl cp pki_intermediate.csr vault-0:/home/vault

kubectl exec -it vault-0 -- /bin/sh
cd /home/vault/ && vault write -format=json pki/root/sign-intermediate csr=@pki_intermediate.csr format=pem_bundle ttl="43800h" > intermediate.cert.pem
exit
kubectl exec vault-0 cat /home/vault/intermediate.cert.pem | jq -r '.data.certificate' > intermediate.cert.pem

kubectl cp intermediate.cert.pem vault-0:/home/vault/

kubectl exec -it vault-0 -- /bin/sh
cd /home/vault/ && vault write pki_int/intermediate/set-signed certificate=@intermediate.cert.pem
exit
```
output
```
-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUbSLWT/COKAz6L1OZfrZNKhCXJMYwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDA0MTkyMjA3MTFaFw0yNTA0
MTgyMjA3NDFaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALZjhOlSYy8h
QrfSPy9SMDhoPV7jV0YNidCZmTlXZNJIieKE1tCNSiUHqcmtvaPN15qDtf/LqieM
cE95eaiPUUU6zcpvEWGBZkJ2laG2X8ue8+KYVLcc0AQTZHbUvhk/DXQ0+wNpDL8J
hfr+uMrCa3cYNEWWDHeI1ca4m2WCFXotYKMaK2ziWE8FeRmOt2YHaa2zPTW90pm9
6g6m9ADl1GKnl+T71S4C/lUcnlSo4Ekch7qR2WPPxief237qRtXIA3UegrohCMPI
WQrsHWqVP2GCbtO3ChV/jOL6amNQSK8YI8URAoDQAphz+nJvYsMo539TXsDskt+1
a3Q1UPiCXYUCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUCSlVDCr9ji34oFlFI3KuYF8InzkwHwYDVR0jBBgwFoAU
PUNLpcLrUSK7NKBbSloA4b944ZgwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
D0XZ2emwI71oBh4JqjXCs/A1w4DEda/O+StaEamAdAr6JOpP0HP0B4ya9xMhKo/A
2BfZtT5MkydAV9Z0n+6OQ6+5isYSQiDBFe8vs4dfSgtjXWmiGD0udnjVsPJUskSJ
bMf+J+ldfUPTxEWK7/+ainfZqy0EArQIG4Znd2gT5C10l2bOQa2KsBLlqLBOH/dr
vS46MMIMaQtF3ovNi6uC/239fh8oLib+gALHkWabTxXb7NHEY+QkePmnOdCF3/bw
tktSbrGcZTl20hDCDTVMTqVYJu9Q9BehWi6Zpy5z+7QO1WAx68SX9wWRXY/AqWEh
G366vqDIbtTIlvTfkUQe2A==
-----END CERTIFICATE-----
```
 - Создадим и отзовем новые сертификаты
Создадим роль для выдачи сертификатов
```
kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
	allowed_domains="example.ru" allow_subdomains=true   max_ttl="720h"
	
Success! Data written to: pki_int/roles/example-dot-ru
```
Создадим и отзовем сертификат
```
kubectl exec -it vault-0 -- vault write pki_int/issue/example-dot-ru common_name="gitlab.example.ru" ttl="24h"
```
output
```
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUbSLWT/COKAz6L1OZfrZNKhCXJMYwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDA0MTkyMjA3MTFaFw0yNTA0
MTgyMjA3NDFaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALZjhOlSYy8h
QrfSPy9SMDhoPV7jV0YNidCZmTlXZNJIieKE1tCNSiUHqcmtvaPN15qDtf/LqieM
cE95eaiPUUU6zcpvEWGBZkJ2laG2X8ue8+KYVLcc0AQTZHbUvhk/DXQ0+wNpDL8J
hfr+uMrCa3cYNEWWDHeI1ca4m2WCFXotYKMaK2ziWE8FeRmOt2YHaa2zPTW90pm9
6g6m9ADl1GKnl+T71S4C/lUcnlSo4Ekch7qR2WPPxief237qRtXIA3UegrohCMPI
WQrsHWqVP2GCbtO3ChV/jOL6amNQSK8YI8URAoDQAphz+nJvYsMo539TXsDskt+1
a3Q1UPiCXYUCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUCSlVDCr9ji34oFlFI3KuYF8InzkwHwYDVR0jBBgwFoAU
PUNLpcLrUSK7NKBbSloA4b944ZgwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
D0XZ2emwI71oBh4JqjXCs/A1w4DEda/O+StaEamAdAr6JOpP0HP0B4ya9xMhKo/A
2BfZtT5MkydAV9Z0n+6OQ6+5isYSQiDBFe8vs4dfSgtjXWmiGD0udnjVsPJUskSJ
bMf+J+ldfUPTxEWK7/+ainfZqy0EArQIG4Znd2gT5C10l2bOQa2KsBLlqLBOH/dr
vS46MMIMaQtF3ovNi6uC/239fh8oLib+gALHkWabTxXb7NHEY+QkePmnOdCF3/bw
tktSbrGcZTl20hDCDTVMTqVYJu9Q9BehWi6Zpy5z+7QO1WAx68SX9wWRXY/AqWEh
G366vqDIbtTIlvTfkUQe2A==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZzCCAk+gAwIBAgIUT/okaJdUyEUII5cm0cascIbXl7IwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIwMDQxOTIyNDkxMloXDTIwMDQyMDIyNDk0MlowHDEaMBgGA1UEAxMRZ2l0
bGFiLmV4YW1wbGUucnUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDN
v4wQNVGE6USv5s0MNQeBiIXuuFWUD7UIpkkxNl1SBpnE3poTD49AfSyG1G0YWqsg
qiliHg8GwmlcEN3yXIJJkXjwYSn6WAvsVRnxvW57V96v2G4DgOZwMa3IJcofbCA3
nCJhFxZK8+DYoCxcfkc5yZMjs26a3w4c3LCBb3jxGcfiKxWeSbtSOdfkf2SumvYY
OTBLu4LMIOi7yLkHU/tZt1IgF6c6j1ebN5XAJNgfWrUjqYCE9rQW4TXtnEOCbzyT
0UhugbTsOBzIUfNctxQ9VVZhVBKob0nWiGTH1C3SqEa2dvuSD/C8A15QO7fu2+n3
QShUZ3iZB5Rqv5Rtv2KPAgMBAAGjgZAwgY0wDgYDVR0PAQH/BAQDAgOoMB0GA1Ud
JQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjAdBgNVHQ4EFgQUCfwBulNasvxQS5Ci
cLLsI1QMtYcwHwYDVR0jBBgwFoAUCSlVDCr9ji34oFlFI3KuYF8InzkwHAYDVR0R
BBUwE4IRZ2l0bGFiLmV4YW1wbGUucnUwDQYJKoZIhvcNAQELBQADggEBAH+PjSLA
OahG0FPcFMORTX1VnuMBZZHQ3t8zIY3FgyQMBrhsOdUVN5BjJY9ap5ejMVsCrIjG
oePkcBWgAAUqmYci4FDsXTR5dgKXRQMBzofgZb6DiGYlakFPrb7gGlpNRprwqeUP
6btlrDXXPC1kN+RCBN1vZ+9Ak6oel+yhCVxGBEEat2PGJ8s9i/2FD6iLjkG3FBCd
5OG7KRzooasnMfSxy2GP3/ZzvtzqzPcaBjqg2jvaSFLLOoQjvtKGUQvBlusaYBU3
2IvbvOqlYfYXJe0pN1rGzqyBXW+0xrWAfrmCpT0V5hmYx6eUJ4YJSVto6P4AObOA
g2Sv9n8HUsNoqCs=
-----END CERTIFICATE-----
expiration          1587422982
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUbSLWT/COKAz6L1OZfrZNKhCXJMYwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhtYXBsZS5ydTAeFw0yMDA0MTkyMjA3MTFaFw0yNTA0
MTgyMjA3NDFaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALZjhOlSYy8h
QrfSPy9SMDhoPV7jV0YNidCZmTlXZNJIieKE1tCNSiUHqcmtvaPN15qDtf/LqieM
cE95eaiPUUU6zcpvEWGBZkJ2laG2X8ue8+KYVLcc0AQTZHbUvhk/DXQ0+wNpDL8J
hfr+uMrCa3cYNEWWDHeI1ca4m2WCFXotYKMaK2ziWE8FeRmOt2YHaa2zPTW90pm9
6g6m9ADl1GKnl+T71S4C/lUcnlSo4Ekch7qR2WPPxief237qRtXIA3UegrohCMPI
WQrsHWqVP2GCbtO3ChV/jOL6amNQSK8YI8URAoDQAphz+nJvYsMo539TXsDskt+1
a3Q1UPiCXYUCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUCSlVDCr9ji34oFlFI3KuYF8InzkwHwYDVR0jBBgwFoAU
PUNLpcLrUSK7NKBbSloA4b944ZgwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
D0XZ2emwI71oBh4JqjXCs/A1w4DEda/O+StaEamAdAr6JOpP0HP0B4ya9xMhKo/A
2BfZtT5MkydAV9Z0n+6OQ6+5isYSQiDBFe8vs4dfSgtjXWmiGD0udnjVsPJUskSJ
bMf+J+ldfUPTxEWK7/+ainfZqy0EArQIG4Znd2gT5C10l2bOQa2KsBLlqLBOH/dr
vS46MMIMaQtF3ovNi6uC/239fh8oLib+gALHkWabTxXb7NHEY+QkePmnOdCF3/bw
tktSbrGcZTl20hDCDTVMTqVYJu9Q9BehWi6Zpy5z+7QO1WAx68SX9wWRXY/AqWEh
G366vqDIbtTIlvTfkUQe2A==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAzb+MEDVRhOlEr+bNDDUHgYiF7rhVlA+1CKZJMTZdUgaZxN6a
Ew+PQH0shtRtGFqrIKopYh4PBsJpXBDd8lyCSZF48GEp+lgL7FUZ8b1ue1fer9hu
A4DmcDGtyCXKH2wgN5wiYRcWSvPg2KAsXH5HOcmTI7Numt8OHNywgW948RnH4isV
nkm7UjnX5H9krpr2GDkwS7uCzCDou8i5B1P7WbdSIBenOo9XmzeVwCTYH1q1I6mA
hPa0FuE17ZxDgm88k9FIboG07DgcyFHzXLcUPVVWYVQSqG9J1ohkx9Qt0qhGtnb7
kg/wvANeUDu37tvp90EoVGd4mQeUar+Ubb9ijwIDAQABAoIBACx1FD24V6oBgY8E
rw/RMZMCvfXsLMm2bya0T6JrndSIipdohiBSP2Du6z0jD4Owq8N+IT0LQ7HxNjTn
jB2/cYOWBGzkgd6PQdxKMzXx+bp7LSoP+Q2Wv7REd8mEqN0LTxbVDPSvuWhutAn6
6jr+KkPJU5rmPaMHSZRg916wiJCH32NdNGKOqW5i3W4vHiEk9ojs96P0OPqFEuFt
kZTacLUnbMYJoBkRDTyqQeI+jFX64HTggk+vvqCvvOhPiJxo2zYhiH4lp7GaDSLY
l3o/AtAeiLPT22eVdejvoEjBicYx07AU44E7qphRcLnZsnO8Lcb5XwxJusCUDVN3
XjtM3sECgYEA5fub5L0b+J1okJJhE54wCiHrcYF15GchvV5r1Fv0vi2TXSL0vLkk
O/kqRhf42ajzUwoeW6zY12bja83/X7gbmLh7lu/MXT7jZtqUF8Y4otJIw1ViqlbE
82QfgT3RuVbydo81iIdI8ykOHDs87NDfIVGdgqR8mlEtCC8hqbsi2i8CgYEA5QYY
K83VquWfS/dEeSfva3wtz0KsNimXk4UgPfK5i6x0VRBmbyqIn5/Ryf+f4WJC5O7z
62hN6D/duhHLHb+04zOhpSKwiWLtXVAk5T6N+IeIksl6k+osHAMhjL2WvN8lOpRB
JkgO2lvWj6qwp9bpgOlWht7jPRyBsnClBt5yxaECgYEAmZD90qoX8FWjSc4EdmYN
a+akYWFxyDa5R8qxs9ZHQWoLWYiK5S23LTi+apl9OFHhUFSUWuC61nJEb2rZGCBG
hR7Cn9G1o5nnnHldo9ZfYRGLNqtgO5++2UrE4rbVwK13L4WG2IiFRKDoZB+GGheK
tqPZyi9taFnF+Q4Hq19i8U8CgYBqrccOiGdxnlnwKbKEeNymBYbVFypeZEtNlOIq
2OgirmbVxLjiKT7Qk8JFxEN12a5tMvz9d21qtG0Uofsp1UVt23v9DsaKiEEmiRFW
kyfQT1mHfkARrC91sOOsyhbvzUVrHuml6kS447Ry4UA+PfYwbQA3/JR+v8uOEbhd
FbsCYQKBgC5RSgZIGhPuNYQvQyTiIqsSqR3IZH/ZmpDMKTY3tSLT9BQmRuYjUU9k
NJmy8K0UnI+BR7rkwNgLc7iTSOWNuYw/Qx0uYfysqmBc9/+3NHmG5ZKw8oIiakkG
sZ66LNj0kBxZUVrJILNc4Vc6vvTKc5s7A2KGhVtkZSKwDdshDvKI
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       4f:fa:24:68:97:54:c8:45:08:23:97:26:d1:c6:ac:70:86:d7:97:b2
```
Отзовем созданный сертификат
```
kubectl exec -it vault-0 -- vault write pki_int/revoke serial_number="4f:fa:24:68:97:54:c8:45:08:23:97:26:d1:c6:ac:70:86:d7:97:b2"

Key                        Value
---                        -----
revocation_time            1587336703
revocation_time_rfc3339    2020-04-19T22:51:43.894561701Z
```
 - Включить TLS
Меняем значения vault-helm/my-values.yaml
```
server
  extraEnvironmentVars: 
    VAULT_ADDR: https://localhost:8200
    VAULT_CACERT: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
  extraVolumes: 
    - type: secret
      name: vault-certs
      path: null # /vault/userconfig
  standalone:
    enabled: "-"
    config: |
      ui = true
      listener "tcp" {
        tls_disable = 1
        address = "[::]:8200"
        cluster_address = "[::]:8201"
      }
      storage "file" {
        path = "/vault/data"
      }
  ha:
    config: |
      ui = true
      cluster_addr = "https://POD_IP:8201"
      listener "tcp" {
        tls_disable = 0
        address = "[::]:8200"
        cluster_address = "[::]:8201"
        tls_cert_file = "/vault/userconfig/vault-certs/tls.crt"
        tls_key_file  = "/vault/userconfig/vault-certs/tls.key"      
      }
      storage "consul" {
        path = "vault"
        address = "HOST_IP:8500"
      }
ui:
  enabled: true
  serviceType: "LoadBalancer"
  serviceNodePort: null
  externalPort: 8200
```
Генерируем rsa ключ
```
openssl genrsa -out vault_gke.key 4096
```
Создаем файл конфигурации
```
nano vault_gke_csr.cnf

# Вставляем
[ req ]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = v3_ext
[ dn ]
commonName = localhost
stateOrProvinceName = Moscow
countryName = RU
emailAddress = no@na.me
organizationName = NoORG
organizationalUnitName = IT
[ v3_ext ]
basicConstraints = CA:FALSE
keyUsage = keyEncipherment,dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[ alt_names ]
DNS.0 = localhost
DNS.1 = vault
```
Генерируем csr
```
openssl req -config vault_gke_csr.cnf -new -key vault_gke.key -nodes -out vault.csr
```
Далее из ресурса - https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
```
cat ./vault.csr | base64 | tr -d '\n'
```
Создаем манифест vault-tls.yaml (описанный на сайте) и применяем его
```
kubectl apply -f vault-https/vault-tls.yaml
```
Подтверждаем использование самоподписанного сертификата
```
kubectl certificate approve vaultcsr
```
Достаем crt из кластера
```
kubectl get csr vaultcsr -o jsonpath='{.status.certificate}'  | base64 --decode > vault.crt
```
Создаем секрет
```
kubectl create secret tls vault-certs --cert=vault.crt --key=vault_gke.key
```
Удаляем развернутый ранее vault для применения протокола https
```
helm delete vault vault-helm
```
Разворачиваем vault с включенным https
```
helm install vault vault-helm -f vault-helm/my-values.yaml
```
Распечатываем vault (раняя инициализация vault), consul хранит все ключи и поды уже инициализированны, но не распечатаны.
Видим, что vault-ui отвечает теперь по https
```
curl http://35.228.235.56:8200/ 
Client sent an HTTP request to an HTTPS server.
```
![vault.https](./vault-https/vault.https.png)
 - Настроить автообновление сертификатов nginx
Собираем docker.
Пишем манифест для nginx и vault-auth
Напишем дополнительную политику для nginx и применем её
```
kubectl cp nginx/nginx-policy.hcl vault-0:/home/vault/

kubectl exec -it vault-0 -- vault policy write nginx /home/vault/nginx-policy.hcl

kubectl exec -it vault-0 -- vault write pki_int/roles/example-dot-ru \
    allowed_domains="example.ru" allow_subdomains=true max_ttl="4m" 
	
kubectl apply -f nginx/deploy.yaml
```
![nginx01](./nginx/nginx01.png)
![nginx02](./nginx/nginx02.png)
## Как запустить проект:
 - Описание выше.

## Как проверить работоспособность:
 - После запуска, можно в кластере проверить.

## PR checklist:
 - [x] Выставлен label с номером домашнего задания.
