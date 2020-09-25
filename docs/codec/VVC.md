# VVC

What is VVC: https://www.streamingmedia.com/Articles/Editorial/Featured-Articles/What-Is-VVC-135215.aspx 

VVC参考软件VTM：https://vcgit.hhi.fraunhofer.de/jvet/VVCSoftware_VTM

```shell
git clone https://vcgit.hhi.fraunhofer.de/jvet/VVCSoftware_VTM.git
git tag
git checkout -b branch_name tag_name
```

部分参数：

```
-q,   --QP
	Qp value
-qpif, --QPIncrementFrame      
	If a source file frame number is specified, the internal QP will be incremented for all POCs associated with source frames >= frame number. If empty, do not increment.
```

