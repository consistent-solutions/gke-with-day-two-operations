# external-dns notes

The automation will create a publicly managed zone with name of {{domain_name}}.

So, create the domain with same name in google cloud dns (or any other domain registrar). Once the domain has been created, be sure to add the nameservers from you managed zone to the nameserver entries in the domain. After a few hours (or could be up to days), traffic should be resolvable



#NOTE on nameservers with managed az -> clouddns

https://cloud.google.com/dns/docs/update-name-servers
https://cloud.google.com/dns/records#adding_or_removing_a_record

(this link led me to the 2 above: https://github.com/openshift/installer/issues/2516)


https://www.cloudbooklet.com/setting-up-google-cloud-dns-for-your-domain/


#NOTE: used these instructions for external-dns - https://github.com/kubernetes-sigs/external-dns/blob/master/docs/tutorials/gke.md


#NOTE: AWESOME material on dns
https://cloud.google.com/dns/docs/dns-overview
