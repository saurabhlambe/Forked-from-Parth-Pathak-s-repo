#### SSL for Solr:
----------------


> source /etc/hadoop/conf/hadoop-env.sh

> ./keytool -genkeypair -alias solr-ssl -keyalg RSA -keysize 2048 -keypass secret -storepass secret -validity 9999 -keystore solr-ssl.keystore.jks

> mv solr-ssl.keystore.jks ~

> ./keytool -importkeystore -srckeystore ~/solr-ssl.keystore.jks -destkeystore ~/solr-ssl.keystore.p12 -srcstoretype jks -deststoretype pkcs12

> openssl pkcs12 -in ~/solr-ssl.keystore.p12 -out ~/solr-ssl.pem

> cat solr-ssl.pem

> openssl pkcs12 -nokeys -in solr-ssl.keystore.p12 -out solr-ssl.cacert.pem

> mkdir solr-ssl-keys

> mv solr-ssl* solr-ssl-keys/

> cd solr-ssl-keys/

> mv solr-ssl-keys /etc/security/

> chown -R solr:hadoop solr-ssl-keys/

> /usr/cloudera-hdp-solr/5.0.0.0-102/cloudera-hdp-solr/solr/server/scripts/cloud-scripts/zkcli.sh -zkhost c4218-node3:2181/solr -cmd clusterprop -name urlScheme -val https
