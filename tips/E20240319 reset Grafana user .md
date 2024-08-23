## For reset admin user

$ docker exec -it grafana bash     
$ grafana-cli --homepath "/usr/share/grafana" admin reset-admin-password "admin"
