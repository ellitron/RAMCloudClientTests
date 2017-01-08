RAMCloud Client Tests
=====================
This is a repository of minimal clients that reproduce bugs found in practice,
usually encountered while running much more complex clients. It currently
includes the following:
* TxMemoryTestCase
  * Description: Executes a read/write transaction on a single object in a 
    loop.
  * Problem: After a few thousand transactions the master server gets stuck
    reporting that it is out of memory. In principle this condition should be
    transient as the log cleaner should free up space.

General Instructions (For All Test Cases)
=========================================
* Edit Makefile
  * Change `RAMCLOUD_DIR` to be correct location
* `make`
* Make sure `$(RAMCLOUD_DIR)/obj.master` is in your `LD_LIBRARY_PATH`. The test
  client dynamically links against the RAMCloud client library.

TxMemoryTestCase Instructions
=============================
* Start a RAMCloud cluster with something like:
```
./scripts/cluster.py -s 1 -r 0 -v --masterArgs="--totalMasterMemory 80%"
```
* Run the MemoryTestClient with the following parameters (use --help for info):
```
./MemoryTestCase -C basic+udp:host=192.168.1.110,port=12246 --tableName bob --serverSpan 1 --size 1024 --txCount 10000
```
* You should see the following output:
```
...
1483833005.315541431 MemoryTestCase.cc:110 in main NOTICE[1]: Executing transaction 805

1483833005.315569671 MemoryTestCase.cc:110 in main NOTICE[1]: Executing transaction 806

1483833005.315597629 MemoryTestCase.cc:110 in main NOTICE[1]: Executing transaction 807

1483833005.315679432 RpcWrapper.cc:220 in isReady NOTICE[1]: Server basic+infud:host=rc01,lid=32,qpn=1067 returned STATUS_RETRY from TX_PREPARE request: Log is out of space! Transaction abort-vote wasn't logged.
1483833010.316025864 RpcWrapper.cc:220 in isReady NOTICE[1]: (3284 duplicates of this message were skipped) Server basic+infud:host=rc01,lid=32,qpn=1067 returned STATUS_RETRY from TX_PREPARE request: Log is out of space! Transaction abort-vote wasn't logged.
1483833010.316025864 ClientLeaseAgent.cc:54 in getLease NOTICE[1]: (696 duplicates of this message were skipped) Blocked waiting for lease to renew.  1483833015.316193006 RpcWrapper.cc:220 in isReady NOTICE[1]: (3258 duplicates of this message were skipped) Server basic+infud:host=rc01,lid=32,qpn=1067 returned STATUS_RETRY from TX_PREPARE request: Log is out of space! Transaction abort-vote wasn't logged.
1483833020.317365024 RpcWrapper.cc:220 in isReady NOTICE[1]: (3253 duplicates of this message were skipped) Server basic+infud:host=rc01,lid=32,qpn=1067 returned STATUS_RETRY from TX_PREPARE request: Log is out of space! Transaction abort-vote wasn't logged.
1483833025.318351744 RpcWrapper.cc:220 in isReady NOTICE[1]: (3265 duplicates of this message were skipped) Server basic+infud:host=rc01,lid=32,qpn=1067 returned STATUS_RETRY from TX_PREPARE request: Log is out of space! Transaction abort-vote wasn't logged.
...
```
* The client stops making progress at this point and continues to output the
  above messages.
* Then when you shut down the cluster, you will see the following output:
```
jdellit@rcmaster:~/RAMCloud$ ./scripts/cluster.py -s 1 -r 0 -v --masterArgs="--totalMasterMemory 80%"
num_servers=(1), available hosts=(10) defined in config.py
disjunct= False

...

All servers running
Servers started.
Type <Enter> to shutdown servers: 
**** server1.rc01.log:
1483833005.315657915 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: Memory capacity exceeded; must delete objects before any more new objects can be created
1483833010.316031530 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (6569 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
1483833015.316200668 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (6517 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
1483833020.317375052 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (6507 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
1483833025.318365799 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (6531 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
1483833030.319230306 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (4388 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
1483833035.320536727 AbstractLog.cc:308 in hasSpaceFor WARNING[6]: (3279 duplicates of this message were skipped) Memory capacity exceeded; must delete objects before any more new objects can be created
**** coordinator.rc10.log:
1483832952.032755748 CoordinatorClusterClock.cc:159 in recoverClusterTime WARNING[1]: couldn't find "coordinatorClusterClock" object in external storage; starting new clock from zero; benign if starting new cluster from scratch, may cause linearizability failures otherwise
1483832952.034486311 CoordinatorUpdateManager.cc:68 in init WARNING[8]: couldn't find "coordinatorUpdateManager" object in external storage; starting new cluster from scratch
```
