### Local Environments
These are local development environments.
You can review the Vagrantfiles if you would like to customize and package the boxes.
You can dowload and use the box with the Vagrantfile.  Development boxes are included in the `boxes` folder.

#### Contents
- [Setup](#setup)
- [Boxes](#boxes)


#### Setup
1. **Vagrant**: Download the (mac) client - [https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)
2. **Virtualbox**: Download and install for (OS X) - [https://www.virtualbox.org](https://www.virtualbox.org)
3. **Ngrok**: Install for sharing - [https://ngrok.com/download](https://ngrok.com/download)
4. Install These Vagrant Plugins
- `vagrant plugin install vagrant-triggers`
- `vagrant plugin install vagrant-hostmanager` 
 

#### Boxes
| Environment | Use Case |
|-------------|----------|
| Nginx       | [Reverse proxy](/nginx) |
| Node        | [Node with Nginx](/node) |
| Express     | Node Web server |
| Loopback    | Node API server |
| Hapi        | Node API server |
| Data        | Redis, Mongo, MySql |




#### Mongo Best Practices
- [MongoDB](https://blog.engineyard.com/2011/mongodb-best-practices)
- [Schema](https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design-part-1)
- [Performance](https://www.sitepoint.com/7-simple-speed-solutions-mongodb/)





