#!/bin/sh

echo "Feature Description"

VARENV=$1

if [[ $(fgrep -ix $VARENV <<< 'qa') ]]; then
  JENKINS_BUILD_JOB_NAME='<JOBNAME>'
else
  JENKINS_BUILD_JOB_NAME='<JOBNAME>'
fi

FILE=/usr/local/bin/.jenkins_api_token

if test -f "$FILE"; then
    echo "$FILE exists."
  else
    echo "FetchUserJenkinsApiDetails"
    echo "$FILE doesnt exists. Creating new temp file"
    echo "Please enter a JENKINS UserName:"
    read VARNAME
    echo "Please enter a JENKINS ApiKey:"
    read VARAPIKEY
    echo "$VARNAME\n$VARAPIKEY" > $FILE
fi

value=`cat $FILE`

SAVEIFS=$IFS   # Save current IFS (Internal Field Separator)
IFS=$'\n'      # Change IFS to newline char
JENKINS_CREDS=($value) # split the `names` string into an array by the same name
IFS=$SAVEIFS   # Restore original IFS

JENKINS_USERNAME=${JENKINS_CREDS[0]}
JENKINS_APIKEY=${JENKINS_CREDS[1]}

ipaddress=$(curl ifconfig.me)

echo "ipaddress, $ipaddress"

OUTPUT="$(curl -H 'Cache-Control: no-cache' -s -i -u $JENKINS_USERNAME':'$JENKINS_APIKEY POST 'https://localhost:8080/jenkins/<path>/job/'$JENKINS_BUILD_JOB_NAME'/buildWithParameters' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'newip='$ipaddress \
--data-urlencode 'desc='$JENKINS_USERNAME | grep -i ^location: | cut -d: -f2- | sed 's/^ *\(.*\).*/\1/')"

JENKINS_BUILD_ROOM_PATH="api/json"
JENKINS_BUILD_ROOM="${OUTPUT/$'\r'/}${JENKINS_BUILD_ROOM_PATH}"

echo "build, ${JENKINS_BUILD_ROOM}"

echo "--------------------------------------------------------------------------"

JSON_VALUE="hudson.model.FreeStyleBuild"

# set the URL to call
URL="${JENKINS_BUILD_ROOM}"

# execute the loop
while true
do
    # make the request
    RESPONSE=$(curl -s -u $JENKINS_USERNAME':'$JENKINS_APIKEY --location --request GET ${JENKINS_BUILD_ROOM})

    # parse the json value
    VALUE=$(echo $RESPONSE | jq -r ".executable._class")

    # compare the values
    if [ "$VALUE" = "$JSON_VALUE" ]; then
        echo "Build Started successfully"
        break
    else
        echo "Build in queue, waiting 2 seconds before next try"
        sleep 2
    fi
done

echo "--------------------------------------------------------------------------"

JENKINS_BUILD_ROOM_OUTPUT="$(curl -s -u $JENKINS_USERNAME':'$JENKINS_APIKEY --location --request GET ${JENKINS_BUILD_ROOM})"


BUILD_URL=$(echo $JENKINS_BUILD_ROOM_OUTPUT | jq -r ".executable.url")

echo "build url, ${BUILD_URL}"

while true
do

  JENKINS_BUILD_ROOM_OUTPUT="$(curl -s -u $JENKINS_USERNAME':'$JENKINS_APIKEY --location --request GET $BUILD_URL'api/json')"


    JENKINS_BUILD_RESUT=$(echo $JENKINS_BUILD_ROOM_OUTPUT | jq -r ".result")

    if [ "$JENKINS_BUILD_RESUT" = null ]; then
        echo "Build inprogress, retry after 2sec"
        sleep 2

      else
        echo "$JENKINS_BUILD_RESUT"
        break;
    fi
done

echo "--------------------------------------------------------------------------"

if [[ "$JENKINS_BUILD_RESUT" = "FAILURE" ]]; then
  JENKINS_BUILD_ROOM_OUTPUT="$(curl -s -u $JENKINS_USERNAME':'$JENKINS_APIKEY --location --request GET $BUILD_URL'consoleText')"

  echo "REASON OF BUILD FAILURE: $BUILD_URL"
  echo "$JENKINS_BUILD_ROOM_OUTPUT"
else
  echo "successful with jenkins build: $BUILD_URL"
fi
