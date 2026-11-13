cert-manager watches the Certificate resource, reads the issuerRef to identify the configured Issuer or ClusterIssuer, uses its configuration to communicate with the CA such as DigiCert
and obtain the certificate, and then stores or updates the certificate and private key in the Kubernetes Secret specified by secretName.
