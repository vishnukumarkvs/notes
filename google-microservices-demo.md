google-microservices-demo
=========================


helm upgrade onlineboutique oci://us-docker.pkg.dev/online-boutique-ci/charts/onlineboutique \
  --install \
  --namespace onlineboutique \
  --create-namespace \
  --set opentelemetryCollector.create=true \
  --set serviceAccounts.create=false
  
  
Notes on  main ☸ default (onlineboutique) ❯ kgp
NAME                                      READY   STATUS             RESTARTS      AGE
adservice-6969db987d-7w75w                0/1     ImagePullBackOff   0             47s
cartservice-77fd569748-5qhhh              0/1     ImagePullBackOff   0             47s
checkoutservice-64b664749f-r5m62          0/1     ImagePullBackOff   0             47s
currencyservice-58dd7648ff-v6mjb          0/1     ErrImagePull       0             46s
emailservice-6769cb5cf5-mltks             0/1     ImagePullBackOff   0             47s
frontend-85d75655c7-mjwcd                 0/1     ImagePullBackOff   0             47s
loadgenerator-b65c549c9-c8dqw             0/1     Init:0/1           0             47s
opentelemetrycollector-8587bf759b-mlzkh   0/1     Error              2 (24s ago)   47s
paymentservice-84d94cc4f8-tvnb9           0/1     ImagePullBackOff   0             47s
productcatalogservice-6bf9db75fc-czx9w    0/1     ImagePullBackOff   0             47s
recommendationservice-5b98bf884d-2mpgs    0/1     ImagePullBackOff   0             46s
redis-cart-7bd75f6fb6-nrlx7               1/1     Running            0             47s
shippingservice-5498484b6-72wv6           0/1     ImagePullBackOff   0             47s

Notes on  main ☸ default (onlineboutique) ❯



old and doesnt work

