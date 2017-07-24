### Local Environments
These are local development environments.
You can review the Vagrantfiles if you would like to customize and package the boxes.
You can dowload and use the box with the Vagrantfile.  Development boxes are included in the `boxes` folder.

#### Contents
- [Requirements](#requirements)
- [Boxes](#boxes)
- [Nginx](#nginx)
- [Express](#express) 
- [Loopback](#loopback)
- [Hapi](#hapi)
- [Data](#data) 


#### Requirements
You must install vagrant, virtualbox, and the below plugins
- `vagrant plugin install vagrant-hostmanager`
- `vagrant plugin install vagrant-triggers`


#### Boxes
| Environment | Use Case |
|-------------|----------|
| Nginx       | Reverse proxy |
| Express     | Node Web server |
| Loopback    | Node API server |
| Hapi        | Node API server |
| Data        | Redis, Mongo, MySql |



#### TO DO
- Add notes on how to install vagrant
- virtualbox
- build other boxes


#### Mongo Best Practices
- [MongoDB](https://blog.engineyard.com/2011/mongodb-best-practices)
- [Schema](https://www.mongodb.com/blog/post/6-rules-of-thumb-for-mongodb-schema-design-part-1)
- [Performance](https://www.sitepoint.com/7-simple-speed-solutions-mongodb/)





