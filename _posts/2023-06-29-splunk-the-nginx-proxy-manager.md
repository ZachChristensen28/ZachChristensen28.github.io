---
layout: post
title: Splunk the Nginx Proxy Manager
categories: [Tutorial]
tags: ["getting data in"]
date: 2023-06-29 20:25 -0600
image:
    path: /assets/img/splunk-nginx/nginx-proxy-manager-splunk.png
    lqip: /assets/img/splunk-nginx/nginx-proxy-manager-splunk.png
    alt: The unofficial guide to getting the Nginx Proxy Manager logs into Splunk.
---

The [Nginx Proxy Manager](https://nginxproxymanager.com/){: target="blank" } is an effortless way to expose services securely. This pre-built docker image enables you to easily forward to your websites running at home or otherwise, including free SSL, without knowing too much about Nginx or Letsencrypt.

## What this guide is

This guide will show you one way to visualize the Nginx proxy data in Splunk.

## What this guide is not

Detailed procedures to install and configure Splunk and the Nginx Proxy Manager. This guide assumes you have an existing Splunk environment and a basic understanding of log collection through a [Splunk Universal Forwarder](https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Abouttheuniversalforwarder){: target="blank" }.

## Resources

- Nginx Proxy Manager documentation: <https://nginxproxymanager.com/guide/>{: target="blank" }
- Deploy a Universal Forwarder: <https://docs.splunk.com/Documentation/Forwarder/latest/Forwarder/Installtheuniversalforwardersoftware>{: target="blank" }

## Getting Started

- Ubuntu Server with the Nginx Proxy Manager deployed using the docker-compose configurations identified in the [quick start guide](https://nginxproxymanager.com/guide/#quick-setup{: target="blank" }). This server also has a Splunk Universal Forwarder installed for data collection.
- Ubuntu Server with Portainer deployed that is managing the Nginx Proxy Manager.

> The Portainer server will not be involved with data collection. It is merely here for context, as some file paths may differ.
{: .prompt-info }

## Prepare Splunk

Splunk will need to have the [add-on for Nginx](https://splunkbase.splunk.com/app/3258){: target="blank" } installed on the Search heads and indexers in the environment. This will give us the necessary field extractions to use these logs correctly.

- Download and install the [Splunk Add-on for Nginx](https://splunkbase.splunk.com/app/3258){: target="blank" } on Search Heads and indexers.

## Locate the Nginx Proxy Manager logs

The log location for the Nginx logs may differ depending on how you set up your docker-compose file. Using the default configuration, the starting path to the log file will be under the /data directory. The exact directory used for this guide's environment is `/data/compose/14/data/logs`.

> Once you locate this directory, ensure the files within have sufficient permissions for Splunk to read.
{: .prompt-tip }

## Update logging configuration for Nginx

> This section is optional but recommended to improve the logging format of Nginx.
{: .prompt-info}

To get the most out of the Nginx logs in Splunk, using the KV format instead of a raw format for the access log is recommended. See [Splunk docs](https://docs.splunk.com/Documentation/AddOns/released/NGINX/Setupv2){: target="blank" } for more information on this format.

On the Nginx Proxy Manager host, navigate to the Nginx configurations directory. The path for this guide's environment is `/data/compose/14/data/nginx`. According to the [Nginx Proxy Manger's docs](https://nginxproxymanager.com/advanced-config/#custom-nginx-configurations){: target="blank" }, custom Nginx configurations should reside in a custom directory.

- Create a new directory called "custom" under the Nginx directory.

```shell
cd /data/compose/14/data/nginx # change to the correct directory
mkdir custom
```

- Create a new file named [`http_top.conf`](https://nginxproxymanager.com/advanced-config/#custom-nginx-configurations){: target="blank" } which will place our configurations at the top of the HTTP block. Place the [recommended log format for Nginx logs](https://docs.splunk.com/Documentation/AddOns/released/NGINX/Setupv2){: target="blank" } into this file.

```nginx
log_format kv 'site="$server_name" server="$host" dest_port="$server_port" dest_ip="$server_addr" '
                       'src="$remote_addr" src_ip="$realip_remote_addr" user="$remote_user" '
                       'time_local="$time_local" protocol="$server_protocol" status="$status" '
                       'bytes_out="$bytes_sent" bytes_in="$upstream_bytes_received" '
                       'http_referer="$http_referer" http_user_agent="$http_user_agent" '
                       'nginx_version="$nginx_version" http_x_forwarded_for="$http_x_forwarded_for" '
                       'http_x_header="$http_x_header" uri_query="$query_string" uri_path="$uri" '
                       'http_method="$request_method" response_time="$upstream_response_time" '
                        'cookie="$http_cookie" request_time="$request_time" category="$sent_http_content_type" https="$https"';
```
{: file="/data/compose/14/data/nginx/custom/http_top.conf" }

### Update the Nginx config to use the new format

1. Create a file named [`server_proxy.conf`](https://nginxproxymanager.com/advanced-config/#custom-nginx-configurations) in the custom directory we created earlier.

    ```shell
    touch server_proxy.conf
    ```

2. Place the following in the new file.

    ```nginx
    access_log /data/logs/all_proxy_access.log kv;
    error_log /data/logs/all_proxy_error.log warn;
    ```
    {: file="/data/compose/14/data/nginx/custom/server_proxy.conf" }

    > Update `all_proxy_access.log` and `all_proxy_error.log` to the file names you want to use.
    {: .prompt-tip }

3. Restart the docker container for the changes to take effect. 

#### Example Output

```shell
site="www.zachthesplunker.com" server="zachthesplunker.com" dest_port="443" 
dest_ip="0.0.0.0" src="0.0.0.0" src_ip="0.0.0.0" user="-" 
time_local="16/Jun/2023:17:58:01 +0000" protocol="HTTP/2.0" 
status="200" bytes_out="5363" bytes_in="5437" http_referer="-" 
http_user_agent="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.97 Safari/537.36" 
nginx_version="1.19.9" http_x_forwarded_for="0.0.0.0" http_x_header="-" 
uri_query="-" uri_path="/" http_method="GET" response_time="0.036" 
cookie="-" request_time="0.038" category="text/html; charset=utf-8" https="on"
```
{: file="/data/logs/all_proxy_access.log" .nolineno }

## Configure Splunk Monitoring

Now that we have logs in the format we want let's get them in Splunk. We will use the Universal Forwarded deployed on the Nginx Proxy Manager to set up monitoring inputs.

Below is an example of configurations needed to ingest these logs—update according to your environment.

```ini
[monitor:///data/compose/14/data/logs/all_proxy_access.log]
disabled = 0
index = web
sourcetype = nginx:plus:kv

[monitor:///data/compose/14/data/logs/all_proxy_error.log]
disabled = 0
index = web
sourcetype = nginx:plus:error
```
{: file="inputs.conf" }

Once these configurations have been deployed to the Universal Forwarder and restarted, you should see logs in Splunk!

## Conclusion

The Nginx Proxy Manager is an easy way to serve internal services securely. Using Splunk, we can gain visibility into the traffic to our web services allowing us to generate detailed reports and alerts based on the activity.