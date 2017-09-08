---
layout: post
title:  "Fixing key issues in Microsoft SQL Server"
date:   2017-09-04 09:04:42 -0400
categories: Tech
---
<h1> Issue </h1>

The reason for this post it to outline my troubleshooting process and solution for the issues I faced after trying to restore a database from a different environment. The purpose of this was to move modern data onto our development environment to allow for more accurate testing.

After restoring the database(DB) there were multiple issues with the Master Key, Symmetrical Key and certificates. A simple solution is to simply delete the keys and recreate the Master Key which would allow the Symmetrical and certs to be recreated. However, the issue was that the Data Warehouse connected to the DB was not allowing the keys to be dropped. 
