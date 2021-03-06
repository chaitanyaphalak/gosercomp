## Golang 序列化反序列化库的性能比较

### 测试的 Serializers

以golang自带的_encoding/json_和_encoding/xml_为基准，测试以下性能比较好的几种序列化库。

- [encoding/json](http://golang.org/pkg/encoding/json/)
- [encoding/xml](http://golang.org/pkg/encoding/xml/)
- [github.com/tinylib/msgp](http://github.com/tinylib/msgp)
- [github.com/golang/protobuf](http://github.com/golang/protobuf)
- [github.com/gogo/protobuf](http://github.com/gogo/protobuf)
- [github.com/google/flatbuffers](http://github.com/google/flatbuffers)
- [Apache/Thrift](https://github.com/apache/thrift/tree/master/lib/go)
- [Apache/Avro](https://github.com/linkedin/goavro)
- [andyleap/gencode](https://github.com/andyleap/gencode)
- [ugorji/go/codec](https://github.com/ugorji/go/tree/master/codec)
- [go-memdump](https://github.com/alexflint/go-memdump)
- [colfer](https://github.com/pascaldekloe/colfer)
- [zebrapack](https://github.com/glycerine/zebrapack)
- [gotiny](https://github.com/niubaoshu/gotiny)
- [github.com/ugorji/go/codec](http://github.com/ugorji/go/codec)
- [hprose-golang](https://github.com/hprose/hprose-golang/tree/master/io)
- [vmihailenco/msgpack.v2](https://github.com/vmihailenco/msgpack)
- [Sereal](https://github.com/Sereal/Sereal)
- [ffjson](https://github.com/pquerna/ffjson)
- [easyjson](https://github.com/mailru/easyjson)
- [jsoniter](https://github.com/json-iterator/go)

### 排除的 Serializers

基于 alecthomas 已有的[测试](https://github.com/alecthomas/go_serialization_benchmarks)，下面的库由于性能的原因没有进行测试。

- [encoding/gob](http://golang.org/pkg/encoding/gob/)
- [github.com/alecthomas/binary](http://github.com/alecthomas/binary)
- [github.com/davecgh/go-xdr/xdr](http://github.com/davecgh/go-xdr/xdr)
- [labix.org/v2/mgo/bson](http://labix.org/v2/mgo/bson)
- [github.com/DeDiS/protobuf](http://github.com/DeDiS/protobuf)
- [gopkg.in/vmihailenco/msgpack.v2](http://gopkg.in/vmihailenco/msgpack.v2)
- [bson](http://github.com/micro/go-bson)

### 测试环境
go version: **1.10**


- 对于 `MessagePack`，你需要安装库以及利用`go generate`生成相关的类:

  ```go
  go get github.com/tinylib/msgp
  go generate
  ```

- 对于`ProtoBuf`,你需要安装[protoc编译器](https://github.com/google/protobuf/releases)，以及protoc库以及生成相关的类：

  ```go
  go get github.com/golang/protobuf
  go generate
  ```

- 对于`gogo/protobuf`,你需要安装库以及生成相关的类：

  ```go
  go get github.com/gogo/protobuf/gogoproto
  go get -u github.com/gogo/protobuf/protoc-gen-gogofaster
  go generate
  ```

- 对于`flatbuffers`,你需要安装[flatbuffers编译器](https://github.com/google/flatbuffers/releases, 以及flatbuffers库：

  ```go
  go get github.com/google/flatbuffers/go
  go generate
  ```

- 对于`thrift`,), 你需要安装[thrift编译器](https://thrift.apache.org/download)以及thrift库：

  ```go
  go get git.apache.org/thrift.git/lib/go/thrift
  go generate
  ```

- 对于`Avro`,你需要安装goavro库：

    ```go
    go get github.com/linkedin/goavro
    go generate
    ```

- 对于`gencode`,你需要安装gencode库,并使用gencode库的工具产生数据对象：

  ```go
  go get github.com/andyleap/gencode
  bin\gencode.exe go -schema=gencode.schema -package gosercomp
  ```

  `gencode`也是一个高性能的编解码库，提供了代码生成工具，而且产生的数据非常的小。

- 对于`easyjson`,你需要安装easyjson库:

  ```go
  go get github.com/mailru/easyjson
  go generate
  ```

- 对于`zebraPack `,你需要安装zebraPack库,并使用zebraPack工具产生数据对象：

  ```go
  go get github.com/glycerine/zebrapack
  go generate zebrapack_data.go 
  ```

- 对于`ugorji/go/codec`,你需要安装代码生成工具和`codec`库:

```go
  go get -tags=unsafe  -u github.com/ugorji/go/codec/codecgen
  go get -tags=unsafe -u github.com/ugorji/go/codec

  codecgen.exe -o data_codec.go data.go
```

`ugorji/go/codec`是一个高性能的编解码框架，支持 msgpack、cbor、binc、json等格式。本测试中测试了 cbor  和 msgpack的编解码，可以和上面的 `tinylib/msgp`框架进行比较。

> 事实上，这里通过`go generate`生成相关的类，你也可以通过命令行生成，请参考`data.go`中的注释。 但是你需要安装相关的工具，如Thrift,并把它们加入到环境变量Path中

**运行下面的命令测试:**

```
go test -bench=. -benchmem
```

### 测试数据

所有的测试基于以下的struct,自动生成的struct， 比如protobuf也和此结构基本一致。
所以本测试的数据以小数据为主， 不同的测试数据(数据大小，数据类型)可能会导致各框架的表现不一样，注意区别。

```go
type ColorGroup struct {
    ID     int `json:"id" xml:"id,attr""`
    Name   string `json:"name" xml:"name"`
    Colors []string `json:"colors" xml:"colors"`
}
`
```

### 性能测试结果

```
BenchmarkMarshalByJson-4                       	 2000000	       870 ns/op	     368 B/op	       3 allocs/op
BenchmarkUnmarshalByJson-4                     	  500000	      2496 ns/op	     344 B/op	       9 allocs/op
BenchmarkMarshalByXml-4                        	  300000	      3857 ns/op	    4800 B/op	      11 allocs/op
BenchmarkUnmarshalByXml-4                      	  100000	     12779 ns/op	    3171 B/op	      75 allocs/op
BenchmarkMarshalByMsgp-4                       	20000000	       109 ns/op	      80 B/op	       1 allocs/op
BenchmarkUnmarshalByMsgp-4                     	10000000	       219 ns/op	      32 B/op	       5 allocs/op
BenchmarkMarshalByProtoBuf-4                   	 3000000	       469 ns/op	     328 B/op	       5 allocs/op
BenchmarkUnmarshalByProtoBuf-4                 	 2000000	       794 ns/op	     400 B/op	      11 allocs/op
BenchmarkMarshalByGogoProtoBuf-4               	20000000	       106 ns/op	      48 B/op	       1 allocs/op
BenchmarkUnmarshalByGogoProtoBuf-4             	 3000000	       411 ns/op	     144 B/op	       8 allocs/op
BenchmarkMarshalByFlatBuffers-4                	 5000000	       373 ns/op	      16 B/op	       1 allocs/op
BenchmarkUnmarshalByFlatBuffers-4              	2000000000	         0.87 ns/op	       0 B/op	       0 allocs/op
BenchmarkUnmarshalByFlatBuffers_withFields-4   	10000000	       156 ns/op	       0 B/op	       0 allocs/op
BenchmarkMarshalByThrift-4                     	 3000000	       430 ns/op	      64 B/op	       1 allocs/op
BenchmarkUnmarshalByThrift-4                   	 1000000	      1264 ns/op	     656 B/op	      11 allocs/op
BenchmarkMarshalByAvro-4                       	 3000000	       536 ns/op	      48 B/op	       6 allocs/op
BenchmarkUnmarshalByAvro-4                     	  500000	      3183 ns/op	    1672 B/op	      62 allocs/op
BenchmarkMarshalByGencode-4                    	30000000	        40.0 ns/op	       0 B/op	       0 allocs/op
BenchmarkUnmarshalByGencode-4                  	10000000	       120 ns/op	      32 B/op	       5 allocs/op
BenchmarkMarshalByUgorjiCodecAndCbor-4         	 2000000	       696 ns/op	     112 B/op	       3 allocs/op
BenchmarkUnmarshalByUgorjiCodecAndCbor-4       	 3000000	       603 ns/op	      48 B/op	       6 allocs/op
BenchmarkMarshalByUgorjiCodecAndMsgp-4         	 2000000	       663 ns/op	     112 B/op	       3 allocs/op
BenchmarkUnmarshalByUgorjiCodecAndMsgp-4       	 2000000	       609 ns/op	      48 B/op	       6 allocs/op
BenchmarkMarshalByUgorjiCodecAndBinc-4         	 2000000	       706 ns/op	     112 B/op	       3 allocs/op
BenchmarkUnmarshalByUgorjiCodecAndBinc-4       	 1000000	      1069 ns/op	     824 B/op	      10 allocs/op
BenchmarkMarshalByUgorjiCodecAndJson-4         	 2000000	       865 ns/op	     112 B/op	       3 allocs/op
BenchmarkUnmarshalByUgorjiCodecAndJson-4       	 2000000	       764 ns/op	      48 B/op	       6 allocs/op
BenchmarkMarshalByEasyjson-4                   	 5000000	       316 ns/op	     128 B/op	       1 allocs/op
BenchmarkUnmarshalByEasyjson-4                 	 3000000	       481 ns/op	      32 B/op	       5 allocs/op
BenchmarkMarshalByFfjson-4                     	 2000000	       930 ns/op	     424 B/op	       9 allocs/op
BenchmarkUnmarshalByFfjson-4                   	 1000000	      1365 ns/op	     480 B/op	      13 allocs/op
BenchmarkMarshalByJsoniter-4                   	 2000000	       761 ns/op	     800 B/op	       5 allocs/op
BenchmarkUnmarshalByJsoniter-4                 	 3000000	       452 ns/op	     112 B/op	       6 allocs/op
BenchmarkUnmarshalByGJSON-4                    	 1000000	      1807 ns/op	     624 B/op	       7 allocs/op
BenchmarkMarshalByGoMemdump-4                  	  300000	      4862 ns/op	    1032 B/op	      30 allocs/op
BenchmarkUnmarshalByGoMemdump-4                	 1000000	      1462 ns/op	    2400 B/op	      12 allocs/op
BenchmarkMarshalByColfer-4                     	50000000	        29.8 ns/op	       0 B/op	       0 allocs/op
BenchmarkUnmarshalByColfer-4                   	10000000	       200 ns/op	      96 B/op	       6 allocs/op
BenchmarkMarshalByZebrapack-4                  	20000000	       278 ns/op	     132 B/op	       0 allocs/op
BenchmarkUnmarshalByZebrapack-4                	 5000000	       244 ns/op	      32 B/op	       5 allocs/op
BenchmarkMarshalByGotiny-4                     	 5000000	       363 ns/op	     144 B/op	       5 allocs/op
BenchmarkUnmarshalByGotiny-4                   	 5000000	       262 ns/op	      88 B/op	       2 allocs/op
BenchmarkMarshalByHprose-4                     	 3000000	       493 ns/op	     210 B/op	       1 allocs/op
BenchmarkUnmarshalByHprose-4                   	 2000000	       641 ns/op	     288 B/op	       9 allocs/op
BenchmarkMarshalBySereal-4                     	 1000000	      2169 ns/op	     792 B/op	      22 allocs/op
BenchmarkUnmarshalBySereal-4                   	 2000000	       720 ns/op	      80 B/op	       6 allocs/op
BenchmarkMarshalByMsgpackV2-4                  	 1000000	      1872 ns/op	     192 B/op	       4 allocs/op
BenchmarkUnmarshalByMsgpackv2-4                	 1000000	      1603 ns/op	     232 B/op	      11 allocs/op
```

多次测试结果差不多。 从结果上上来看， **MessagePack**,**gogo/protobuf**,和**flatbuffers**差不多，这三个优秀的库在序列化和反序列化上各有千秋，而且都是跨语言的。 从便利性上来讲，你可以选择**MessagePack**和**gogo/protobuf**都可以，两者都有大厂在用。 flatbuffers有点反人类，因为它的操作很底层，而且从结果上来看，序列化的性能要差一点。但是它有一个好处，那就是如果你只需要特定的字段， 你无须将所有的字段都反序列化。从结果上看，不反序列化字段每个调用只用了9.54纳秒，这是因为字段只有在被访问的时候才从byte数组转化为相应的类型。 因此在特殊的场景下，它可以提高N被的性能。但是序列化的代码的面相太难看了。

新增加了**gencode**的测试，它表现相当出色，而且生成的字节也非常的小。

**Codec**的Unmarshal性能不错，但是Marshal性能不是太好。

**colfer**的性能也不错，它能够跨Go，Java, Javascript平台。

新加入的**zebrapack**性能抢眼，不但性能卓越，而且可以实现zero allocation，值得关注。

## 序列化大小

下面显示了各个序列化相同的数据后的大小：

```
	gosercomp_test.go:91: json:				 65 bytes
	gosercomp_test.go:94: xml:				 137 bytes
	gosercomp_test.go:97: msgp:				 47 bytes
	gosercomp_test.go:100: protobuf:				 36 bytes
	gosercomp_test.go:103: gogoprotobuf:			 36 bytes
	gosercomp_test.go:107: flatbuffers:			 108 bytes
	gosercomp_test.go:113: thrift:				 63 bytes
	gosercomp_test.go:127: avro:				 32 bytes
	gosercomp_test.go:136: gencode:				 34 bytes
	gosercomp_test.go:142: UgorjiCodec_Cbor:		 47 bytes
	gosercomp_test.go:148: UgorjiCodec_Msgp:		 47 bytes
	gosercomp_test.go:154: UgorjiCodec_Bin:			 53 bytes
	gosercomp_test.go:156: UgorjiCodec_Json:		 91 bytes
	gosercomp_test.go:159: easyjson:			 65 bytes
	gosercomp_test.go:162: ffjson:				 65 bytes
	gosercomp_test.go:165: jsoniter:			 65 bytes
	gosercomp_test.go:169: memdump:				 200 bytes
	gosercomp_test.go:172: colfer:				 35 bytes
	gosercomp_test.go:175: zebrapack:			 35 bytes
	gosercomp_test.go:178: gotiny:				 32 bytes
	gosercomp_test.go:183: hprose:				 32 bytes
	gosercomp_test.go:187: sereal:				 76 bytes
	gosercomp_test.go:190: msgpackv2:			 47 bytes
```