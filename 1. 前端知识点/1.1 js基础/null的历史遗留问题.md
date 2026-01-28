typeof null 为 object
因为最早设计的null以32为二进制存储，前三位被当成是对象标识

通过`Object.prototype.toString.call(null)` 则可以判断出[object null],可以精确判断一个值是否为null