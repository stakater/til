# Purpose of domain filters

Domain filter in external-dns allow you to keep selected zones synchronized with Ingresses and Services of type = LoadBalancer. We need to make sure never to run multiple external-dns instances with same or repetitive domain filters otherwise it will mess up Route 53 entries.