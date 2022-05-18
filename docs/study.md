

LevelDB默认使用的是小端字节序存储，低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。 
编码分为变长的EncodeVarint和固定大小的EncodeFixed两种，每种又分32位和64位。

    Varint 编码:数字 value

    extern void PutVarint32(std::string* dst, uint32_t value);
    extern void PutVarint32Varint32(std::string* dst, uint32_t value1,
                                    uint32_t value2);
    extern void PutVarint32Varint32Varint32(std::string* dst, uint32_t value1,
                                            uint32_t value2, uint32_t value3);
    extern void PutVarint64(std::string* dst, uint64_t value);
    extern void PutVarint64Varint64(std::string* dst, uint64_t value1,
                                    uint64_t value2);
    extern void PutVarint32Varint64(std::string* dst, uint32_t value1,
                                    uint64_t value2);
    extern void PutVarint32Varint32Varint64(std::string* dst, uint32_t value1,
                                            uint32_t value2, uint64_t value3);

    extern bool GetVarint32(Slice* input, uint32_t* value);
    extern bool GetVarint64(Slice* input, uint64_t* value);
    extern bool GetVarsignedint64(Slice* input, int64_t* value);
    extern const char* GetVarint32Ptr(const char* p,const char* limit, uint32_t* v);
    extern const char* GetVarint64Ptr(const char* p,const char* limit, uint64_t* v);
    inline const char* GetVarsignedint64Ptr(const char* p, const char* limit,
                                                int64_t* value)

    LevelDB 内部采用变长编码，对数据进行压缩，减少存储空间，再采用 CRC 校验数据。

    整型数据是以 32(64) 位来表示的，以 32 位为例，存储需要 4 个字节。

    如果一个整数的大小在 256 以内，那么只需要一个字节就可以存储这个整数，可以节省 3 个字节。

    Varint 就是根据这种思想来序列化整数的，它是一种使用一个或多个字节序列化整数的方法，会把整型数据编码为变长字节。

    Varint 中的每个字节都设置为最高有效位：

    如果该位为 0，表示结束，当前字节的剩余 7 位就是该数据的表示。

    表示整数 1，需要一个字节：0000 0001
    如果该位为 1，表示后续的字节也是该整型数据的一部分；

    表示整数 300，需要两个字节：1010 1100 0000 0010
    这也表示 Varint 编码后是按小端排序的。

    字节顺序，又称端序或尾序（英语：Endianness），在计算机科学领域中，指电脑内存中或在数字通信链路中，组成多字节的字的字节的排列顺序。

    字节的排列方式有两个通用规则。例如，将一个多位数的低位放在较小的地址处，高位放在较大的地址处，则称小端序；反之则称大端序。在网络应用中，字节序是一个必须被考虑的因素，因为不同机器类型可能采用不同标准的字节序，所以均按照网络标准转化。

    因此，32 位整型数据经过 Varint 编码后占用 1～5 个字节（5 * 8 - 5 > 32)，64 位整型数据编码后占用 1~10 个字节（10 * 8 - 10 > 64)。

    在实际场景中，由于小数字的使用率远远高于大数字，所以在大部分场景中，通过 Varint 编码的数据都可以起到很好的压缩效果。
