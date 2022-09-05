https://github.com/docker-mailserver/docker-mailserver#usage

DMS_GITHUB_URL='https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master'
wget "${DMS_GITHUB_URL}/docker-compose.yml"
wget "${DMS_GITHUB_URL}/mailserver.env"

https://github.com/docker-mailserver/docker-mailserver#create-a-docker-compose-environment

docker-compose up
docker-compose up -d

docker exec -ti <CONTAINER NAME> setup email add/del <user@domain>

注意：letsencrept 的证书可能用 sym link，docker 里和 host 里的链接需要完全相同，volume 的文件夹需要包含 link 文件和原文件

https://app.sendinblue.com/
