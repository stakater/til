# SonarQube Authentication with Keycloak

* SonarQube needs this plugin to authenticate via OpenID connect identity provider: https://github.com/vaulttec/sonar-auth-oidc
 (Already installed in stakater sonarqube image)
* The `Admin` credentials and properties needed for this plugin to work can be set through UI or REST api after sonarqube has started. The following post script can be used as kubernetes lifecycle post script: 

    ```
    #!/bin/bash
    set -eo pipefail

    HOME=${HOME:-/opt/app}

    retries=${RETRY_LIMIT:-20};
    SONARQUBE_URL=${SONARQUBE_URL:-http://localhost:9000}
    SETTINGS_PROPERTIES_PATH=${SETTINGS_PROPERTIES_PATH:-"${HOME}/settings/settings.properties"}
    ADMIN_PASSWORD=${ADMIN_PASSWORD:-"admin"}

    function postSetting() {
        local key=$1
        local value=$2
        echo "Updating setting: '${key}'"
        status=$(curl -s -o /dev/null -w "%{http_code}" -XPOST "${SONARQUBE_URL}/api/settings/set?key=${key}&value=${value}" --user admin:${ADMIN_PASSWORD})
        echo "Response Status: ${status}"
        if [ $status -lt 200] || [ $status -gt 299 ]
        then
            echo "Could not update '${key}'"
            exit 1
        fi
    }

    # Wait for SONARQUBE to start up
    until $(curl -s -f -o /dev/null --connect-timeout 1 -m 1 --head "${SONARQUBE_URL}/api/server/version"); do
        echo "Waiting for SonarQube to startup ..."
        sleep 3;
        retries=$(($retries-1))
        
        if [[ ${retries} -eq 0 ]]; 
        then 
            echo "Cannot connect to Sonarqube at ${SONARQUBE_URL}: Retry limit reached";
            exit 1;
        fi
    done

    echo "Sonarqube running at ${SONARQUBE_URL}"
    echo "Waiting for 10 seconds ..."
    sleep 10s

    # Update admin password
    echo "Updating Admin Password for SonarQube"
    pw_status=$(curl -s -o /dev/null -w "%{http_code}" -XPOST --user admin:admin  \
        "${SONARQUBE_URL}/api/users/change_password?login=admin&previousPassword=admin&password=${ADMIN_PASSWORD}")

    echo "Response Status: ${pw_status}"
    echo ""

    # Post settings 
    if [ -f ${SETTINGS_PROPERTIES_PATH} ];
    then
        echo "Posting settings from properties file to SonarQube on ${SONARQUBE_URL}"
        IFS=$'\r\n' GLOBIGNORE='*' command eval  'props=($(cat ${SETTINGS_PROPERTIES_PATH}))'
        for (( i=0; i<${#props[@]}; i++ ))
        do
            IFS='=' read -ra keyValue <<< "${props[$i]}"
            key=${keyValue[0]}
            value=${keyValue[1]}
            postSetting ${key} ${value}
        done
    fi

    # Explicitly set OIDC props from env varaibles
    if [ ! -z "${OIDC_CLIENT_ID}" ];
    then 
        postSetting "sonar.auth.oidc.clientId.secured" ${OIDC_CLIENT_ID}
    fi

    if [ ! -z "${OIDC_CLIENT_SECRET}" ];
    then 
        postSetting "sonar.auth.oidc.clientSecret.secured" ${OIDC_CLIENT_SECRET}
    fi
    ``` 

    Sensitive information can be mounted as environment variables from secrets and the rest can be mounted at `SETTINGS_PROPERTIES_PATH` as a properties file from a config map.


#### NOTE
* If `SONARQUBE_URL` is set through `exposecontroller`'s way of exposing URL, it will be an `http` URL, make sure port 80 is whitelisted on internal-ingress when trying to login.