google-microservices-demo
=========================


helm upgrade onlineboutique oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique \
  --install \
  --namespace onlineboutique \
  --create-namespace \
  --set opentelemetryCollector.create=true \
  --set serviceAccounts.create=false
  
  
