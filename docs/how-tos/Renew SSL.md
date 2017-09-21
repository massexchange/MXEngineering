**Every year, around September 1st, our SSL key and certs need to renewed and rotated. To do purchase and submit the new keys:**

- Go to [CheapSSLSecurity - Comodo PositiveSSL](https://cheapsslsecurity.com/). At the time of writing, they are our SSL certificate vendor.

- Use their CSR generation form to create a CSR + private key. DO NOT FORGET TO COPY/PASTE THEIR GENERATED PRIVATE KEY INTO A FILE.

- Submit the generated CSR to create the certs.

- Verify domain ownership by sending verification email to info@massexchange.com, and following the instructions in the email.

- Once you have been granted the new SSL Private Key and the bundle of **4** .crt files, submit these files to Ops via flash drive or GMail. DO NOT, UNDER ANY CIRCUMSTANCES, SUBMIT THEM THROUGH ANY OTHER MEDIA, INCLUDING HIPCHAT.

**Instructions for Ops:**

- Once you have the key/cert bundle, append the certificates together in order to establish a chain of trust, starting with our root wildcard certificate, and working down the chain of trust, ending with the external root certificate. This can be done very easily using (with Sept. 2017 filenames):

```bash
touch mxsslCombinedCertSept20.crt
cat STAR_massexchange_com.crt >> mxsslCombinedCertSept20.crt
cat COMODORSADomainValidationSecureServerCA.crt >> mxsslCombinedCertSept20.crt
cat COMODORSAAddTrustCA.crt >> mxsslCombinedCertSept20.crt
cat AddTrustExternalCARoot.crt >> mxsslCombinedCertSept20.crt
```

- Then, make any Terraform-deployer environment variable changes necessary in order to have the new keys/certs used in the next deployment.

- Lastly, redeploy our instances with the new keys using 'terraform apply.' Alternatively, you can just place the new keys directly on already-deployed instances wherever MXWeb's configuration says they need to go.
