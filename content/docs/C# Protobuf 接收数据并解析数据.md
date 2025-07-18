# C# Protobuf 接收数据并解析数据

在流里读出数据后，需要将数据包大小缩短成有效大小，即 `bytesRead`  长度。

下面是 C# 端接收并解析数据的示例代码：

```C#
private void ReceiveMessage()
{
    stream = client.GetStream();
    int bytesRead;
    while (isKeepReading)
    {
        if (stream.CanRead)
        {
            buffer = new byte[client.ReceiveBufferSize];
            bytesRead = stream.Read(buffer, 0, client.ReceiveBufferSize);
            if (bytesRead > 0)
            {
                MemoryStream protoStream = new MemoryStream(buffer,0, bytesRead);
                Cas.Proto.DataMessage dataMessage = Cas.Proto.DataMessage.Parser.ParseFrom(protoStream);
                switch (dataMessage.Type)
                {
                    case DataMessage.Types.Type.Mesh:
	                    	Debug.Log("Mesh类型");
                        break;
                    default:
                        break;
                }
            }
        }
    }
}
```

