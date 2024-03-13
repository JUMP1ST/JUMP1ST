# Recommended Network Architecture

The recommended network architecture for deploying Keycloak is to set up an HTTP/HTTPS load balancer on a public IP address that routes requests to Keycloak servers sitting on a private network. This isolates all clustering connections and provides a nice means of protecting the servers.

| Note | By default, there is nothing to prevent unauthorized nodes from joining the cluster and broadcasting multicast messages. This is why cluster nodes should be in a private network, with a firewall protecting them from outside attacks. |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
