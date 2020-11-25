* These notes are the thoughts that went into creating the actual posts and are not part of the website. *

[//]: # (Author: Shebuel Asher Paul, Date: 1st June 2020)

So I want to do a complete post on everything that relates to Authentication and Authorization and while studying for the topic I realized how wide of a topic that is and that it is probably impossible to cover everything in one post. This post will cover as much as I can and any topic that I think requires much more explanation I will create seperate posts for. I basically wanted to do a post on this because I was studying for my cysa+ and realized that I have absolutely no idea of any of these commonly used technologies.


These notes are going to be weird coz they are more of me typing my thoughts which are going to organized when I get started with the post. Just a warning.

## LDAP:

So what is LDAP? LDAP stands for light weight directory access protocol. It is a protocol that lets an user access the directory services through Internet Protocol (IP) TCP/IP stack. 

So what is directory services then? Well b4 we get to directory services we need to understand what a directory is. I actually need to check if a directory in term of LDAP is even a valid term. Anyways I consider a directory as a tree where each of the leaf node is a resource which can be a 

- Computer
- User
- Printer
- Service

operating in the network? 

Each of the node has zero or more attributes attached to it which provide additional detail to the node

So if directory service is just a database of all the network resources in the form of a tree y not just use a regular database? SQL or no SQL?

Well the problem with relational database is that it does not scale well for this functionality coz each resource can have variable number of attributes attached to it.

Well what about no sql then? It does solve the problem of having variable attributes but it brings an additional issue of compatibility? I remember it was somewhere along the lines of it making moving btw different vendors a little difficult.

More than that LDAP uses X.500 which is a well defined protocol that was exclusively built for this purpose which is not the case with relational databases and no sql.

## Basics of LDAP:

LDAP is a protocol that is used to access and maintain the X.500 directory service. The tree which contains all the network resource information is called the Directory Information Tree (DIT).

- Entry:
	Each node in the DIT is called an entry. Each entry consists of a set of attributes, a unique distinguished name (dn) and object classes.

- Distinguished name:
	Each entry has a distinguished name that uniquely identifies that specific entry.  


