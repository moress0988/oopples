name: IBM Cloud Foundry - V2Ray_US

env:
  IBM_CF_API: https://api.eu-gb.cf.cloud.ibm.com
  IBM_CF_APP_MEM: 64M

on:
  workflow_dispatch:
  repository_dispatch:
  schedule:
    - cron: 22 12 * * 2

jobs:
  deploy:
    runs-on: ubuntu-latest
#ubuntu-latest ubuntu-18.04
    timeout-minutes: 5
    env:
      IBM_CF_USERNAME_US: ${{ secrets.IBM_CF_USERNAME_US }}
      IBM_CF_PASSWORD_US: ${{ secrets.IBM_CF_PASSWORD_US }}
      IBM_CF_ORG_NAME_US: ${{ secrets.IBM_CF_ORG_NAME_US }}
      IBM_CF_SPACE_NAME_US: ${{ secrets.IBM_CF_SPACE_NAME_US }}
      IBM_CF_APP_NAME_US: ${{ secrets.IBM_CF_APP_NAME_US }}
      Xray_UUID: ${{ secrets.Xray_UUID }}
      V2_WS_PATH_VMESS: ${{ secrets.V2_WS_PATH_VMESS }}
      Xray_WS_PATH_VLESS: ${{ secrets.Xray_WS_PATH_VLESS }}

    steps:

    - name: Install Cloud Foundry CLI
      run: |
        curl -fsSL "https://packages.cloudfoundry.org/stable?release=linux64-binary&version=v7&source=github" | tar -zxC /tmp
        sudo install -m 755 /tmp/cf7 /usr/local/bin/cf
        cf version
    - name: Login to IBM Cloud Foundry
      run: |
        cf login \
          -a "${IBM_CF_API}" \
          -u "${IBM_CF_USERNAME_US}" \
          -p "${IBM_CF_PASSWORD_US}" \
          -o "${IBM_CF_ORG_NAME_US:-$IBM_CF_USERNAME_US}" \
          -s "${IBM_CF_SPACE_NAME_US:-dev}"
    - name: Download Latest V2Ray
      run: |
        DOWNLOAD_URL="https://github.com/v2fly/v2ray-core/releases/latest/download/v2ray-linux-64.zip"
        curl -fsSL "$DOWNLOAD_URL" -o "latest-v2ray.zip"
        unzip latest-v2ray.zip v2ray v2ctl geoip.dat geosite.dat
        rm latest-v2ray.zip
        chmod -v 755 v2*
        ./v2ray -version
        mv v2ray ${IBM_CF_APP_NAME_US}
    - name: Generate Xray Config File (VLESS)
      if: ${{ env.Xray_WS_PATH_VLESS }}
      run: |
        base64 << P3TER > config
        {
          "log": {
            "access": "none",
            "loglevel": "error"
          },
          "inbounds": [
            {
              "port": 8080,
              "protocol": "vless",
              "settings": {
                "decryption": "none",
                "clients": [
                  {
                    "id": "${Xray_UUID}"
                  }
                ]
              },
              "streamSettings": {
                "network":"ws",
                "wsSettings": {
                  "path": "${Xray_WS_PATH_VLESS}"
                }
              }
            }
          ],
          "outbounds": [
            {
              "protocol": "freedom"
            }
          ]
        }
        P3TER
    - name: Generate Manifest File
      run: |
        cat << P3TER > manifest.yml
        applications:
        - name: ${IBM_CF_APP_NAME_US}
          memory: ${IBM_CF_APP_MEM}
          random-route: true
          command: base64 -d config > config.json; ./${IBM_CF_APP_NAME_US} -config=config.json
          buildpacks:
          - binary_buildpack
        P3TER
    - name: Deploy Cloud Foundry App
      run: cf push
