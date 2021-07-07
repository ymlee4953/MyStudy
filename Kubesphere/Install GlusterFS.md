# Install Gluster FS

1. edit host files for all servers

    - 3 nodes

          vi /etc/hosts

          169.56.76.101	glusterfs-1.d0.k8s.demo.ymlee
          169.56.76.111	glusterfs-2.d0.k8s.demo.ymlee
          169.56.76.123	glusterfs-3.d0.k8s.demo.ymlee

2. install 
    - 3 nodes
    
          yum install centos-release-gluster
          yum install glusterfs-server

-------------------------------------------------------------------------------------------------
- ref: https://github.com/adshafqat/glusterfs/blob/master/heketi-installation.txt
- ref: https://access.redhat.com/documentation/en-us/red_hat_gluster_storage/3.1/html/administration_guide/ch06s02
- ref: https://bryan.wiki/283
- ref: https://computingforgeeks.com/setup-glusterfs-storage-with-heketi-on-centos-server/



      yum install centos-release-gluster -y
      yum install epel-release -y
      yum install heketi -y
      yum install heketi-client -y




        ssh-keygen -f /etc/heketi/heketi_key -t rsa -N ''
        ssh-copy-id -i /etc/heketi/heketi_key root@169.56.76.101
        ssh-copy-id -i /etc/heketi/heketi_key root@169.56.76.111
        ssh-copy-id -i /etc/heketi/heketi_key root@169.56.76.123
        chown heketi:heketi /etc/heketi/heketi_key*


        cat <<EOF> /etc/heketi/topology.json 
        {
        "clusters": [
            {
            "nodes": [
                {
                "node": { "hostnames": { "manage": [ "gfs-server1" ], "storage": [ "192.168.0.200" ] }, "zone": 1 }, "devices": [ "/dev/vdb", "/dev/vdc" ] }, { "node": { "hostnames": { "manage": [ "gfs-server2" ], "storage": [ "192.168.0.201" ] }, "zone": 1 }, "devices": [ "/dev/vdb", "/dev/vdc" ] }, { "node": { "hostnames": { "manage": [ "gfs-server3" ], "storage": [ "192.168.0.202" ] }, "zone": 1 }, "devices": [ "/dev/vdb", "/dev/vdc" ] } ] } ] }

        cat <<EOF> /etc/heketi/topology.json
        {
        "clusters" : [
            {
            "nodes": [
                {
                "node": {
                    "hostnames": {
                    "manage": [
                        "glusterfs-1"
                    ],
                    "storage": [
                        "169.56.76.101"    
                    ]  
                    },
                    "zone": 1
                },
                "device": [
                    "/dev/xvdc"  
                ]
                }, 
                {
                "node": {
                    "hostnames": {
                    "manage": [
                        "glusterfs-2"
                    ],
                    "storage": [
                        "169.56.76.111"    
                    ]  
                    },
                    "zone": 1
                },
                "device": [
                    "/dev/xvdc"  
                ]
                },
                {
                "node": {
                    "hostnames": {
                    "manage": [
                        "glusterfs-3"
                    ],
                    "storage": [
                        "169.56.76.123"    
                    ]  
                    },
                    "zone": 1
                },
                "device": [
                    "/dev/xvdc"  
                ]
                },   
            ]
            }  
        ]    
        }
        EOF
        


        heketi-cli topology load --user admin --secret K8sadmin@ds --json=topology.json

