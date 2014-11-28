OpenTSDB/MapR-DB installation script
====================================

The goal of this script is to make it easier to run OpenTSDB on a MapR-DB cluster.
The only requirements is to have mapr-client (or mapr-core) installed and to install OpenTSDB
either by downloading the latest .rpm/.deb or by building from source then calling 'make install' from the build folder.

Installation instructions on MapR 3.1.1
- build opentsdb from source
	git clone https://github.com/OpenTSDB/opentsdb.git
	cd opentsdb
	./build.sh
- install opentsdb
	cd build
	sudo make install
	sudo ln -s /usr/local/share/opentsdb/etc/opentsdb /etc/opentsdb
	(we may want to create /var/log/opentsdb folder and give read/write access to the user who will launch tsdb "sudo chown mapr:mapr /var/log/opentsdb")
- edit /etc/opentsdb/opentsdb.conf
	tsd.http.staticroot = /usr/local/share/opentsdb/static/
	tsd.core.plugin_path = /usr/local/share/opentsdb/plugins
	tsd.storage.enable_compaction = false
	tsd.storage.hbase.data_table = /user/mapr/tsdb
	tsd.storage.hbase.uid_table = /user/mapr/tsdb-uid
	tsd.storage.hbase.zk_quorum = localhost:5181
- create the tables
	COMPRESSION=NONE \ 
	HBASE_HOME=/opt/mapr/hbase/hbase-0.94.21 \ 
	TSDB_TABLE=/user/mapr/tsdb \ 
	UID_TABLE=/user/mapr/tsdb-uid \ 
	TREE_TABLE=/user/mapr/tsdb-tree \ 
	META_TABLE=/user/mapr/tsdb-meta \ 
	./create_table.sh
		(MapR 4.0.1 you can just run create_table.sh be found in /usr/local/share/opentsdb/tools/
		but in MapR 3.1.1 there is a bug associated with COMPRESSION, run create_table.sh that comes with these instructions instead)
- run "sudo ./install.sh" to download and copy all the missing to opentsdb lib folder
- you can now test the installation:
	tsdb import tmp_import --auto-metric
	tsdb scan --import 1y-ago sum mymetric.stock
