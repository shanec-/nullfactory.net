---
layout: post
title: Could Not Establish Trust Relationship for SSL/TLS Secure Channel  
category: Security
published: draft
---

A while back I worked on a project that required me to integrate to a third-party web service. The web service also in development in parallel by the external team and our team was provided a development endpoint that would be used for testing. 

The problem was the certificate used in the SSL was the same as the one production. This resulted in any call to the web service throwing an `Could not establish trust relationship for SSL/TLS secure channel` error because of the url mismatch. 

Due to various constraints we were unable to get certificate replaced. So our temporary work around was to make our code to explicitly trust the external web service host:

<!--excerpt-->

	using System.Linq;
	using System.Net;
	using System.Net.Security;
	
	namespace SslTest
	{
	    public static class SslHelper
	    {
	        /// <summary>
	        /// Explicitly trust the list of hosts provided and ignores any SSL trust related errors.
	        /// </summary>
	        /// <param name="hosts">List of hosts that are trusted</param>
	        /// <remarks>Only to be used in development environment. Do not use in production!</remarks>
	        public static void EnableTrustedHosts(string[] hosts)
	        {
	            ServicePointManager.ServerCertificateValidationCallback =
	                (sender, certificate, chain, errors) =>
	                {
	                    if (errors == SslPolicyErrors.None)
	                    {
	                        return true;
	                    }
	
	                    var request = sender as HttpWebRequest;
	                    if (request != null)
	                    {
	                        return hosts.Contains(request.RequestUri.Host);
	                    }
	
	                    return false;
	                };
	        }
	    }
	}

The above callback invoked every time a certificate is validated. Call it once before making requests to the web service.

## References
- http://stackoverflow.com/questions/703272/could-not-establish-trust-relationship-for-ssl-tls-secure-channel-soap