# Airflow - First DAGs

### Reference
* bitnami/airflow
- https://github.com/bitnami/charts/tree/master/bitnami/airflow
- https://humbledude.github.io/blog/2019/07/12/airflow-on-k8s/

### issues
#### DAG을 추가하는 방법 확인 필요
- 현재 아래 단계까지 진행 중 : Git repo를 추가하여 DAG 가져오기 
- https://github.com/bitnami/charts/tree/master/bitnami/airflow#load-dag-files
- 아래 페이지에 dag 저장소 구정 내용 참고
- https://humbledude.github.io/blog/2019/07/12/airflow-on-k8s/

#### helm upgrade issue
- airflow helm package를 upgrade 진행 시 error 발생
- upgrade 할 때에는 redis password가 반드시 필요하다고 함
- 정확히 어떤 값들의 지정이 필요한지 확인 필요
> An error occurred: Unable to update the installed package for the package "airflow" using the plugin "helm.packages": rpc error: code = Internal desc = Unable to upgrade helm release "airflow" in the namespace "airflow": Unable to upgrade the release: template: airflow/templates/NOTES.txt:123:4: executing "airflow/templates/NOTES.txt" at <include "common.errors.upgrade.passwords.empty" (dict "validationErrors" $passwordValidationErrors "context" $)>: error calling include: template: airflow/charts/common/templates/_errors.tpl:21:48: executing "common.errors.upgrade.passwords.empty" at <fail>: error calling fail: PASSWORDS ERROR: You must provide your current passwords when upgrading the release. Note that even after reinstallation, old credentials may be needed as they may be kept in persistent volume claims. Further information can be obtained at https://docs.bitnami.com/general/how-to/troubleshoot-helm-chart-issues/#credential-errors-while-upgrading-chart-releases 'auth.fernetKey' must not be empty, please add '--set auth.fernetKey=$AIRFLOW_FERNETKEY' to the command. To get the current value: export AIRFLOW_FERNETKEY=$(kubectl get secret --namespace "airflow" airflow -o jsonpath="{.data.airflow-fernetKey}" | base64 --decode) 'auth.secretKey' must not be empty, please add '--set auth.secretKey=$AIRFLOW_SECRETKEY' to the command. To get the current value: export AIRFLOW_SECRETKEY=$(kubectl get secret --namespace "airflow" airflow -o jsonpath="{.data.airflow-secretKey}" | base64 --decode)

