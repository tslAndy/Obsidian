
```cs
using Cysharp.Text; // namespace

async void Example(int x, int y, int z)
{
    // same as x + y + z
    _ = ZString.Concat(x, y, z);

    // also can use numeric format strings
    _ = ZString.Format("x:{0}, y:{1:000}, z:{2:P}",x, y, z);

    _ = ZString.Join(',', x, y, z);

    // for Unity, direct write(avoid string allocation completely) to TextMeshPro
    tmp.SetTextFormat("Position: {0}, {1}, {2}", x, y, z);

    // create StringBuilder
    using(var sb = ZString.CreateStringBuilder())
    {
        sb.Append("foo");
        sb.AppendLine(42);
        sb.AppendFormat("{0} {1:.###}", "bar", 123.456789);

        // and build final string
        var str = sb.ToString();

        // for Unity, direct write to TextMeshPro
        tmp.SetText(sb);

        // write to destination buffer
        sb.TryCopyTo(dest, out var written);
    }

    // prepare format, return value should store to field(like RegexOptions.Compile)
    var prepared = ZString.PrepareUtf16<int, int>("x:{0}, y:{1:000}");
    _ = prepared.Format(10, 20);

    // C# 8.0, Using declarations
    // create Utf8 StringBuilder that build Utf8 directly to avoid encoding
    using var sb2 = ZString.CreateUtf8StringBuilder();

    sb2.AppendFormat("foo:{0} bar:{1}", x, y);

    // directly write to steam or dest to avoid allocation
    await sb2.WriteToAsync(stream);
    sb2.CopyTo(bufferWritter);
    sb2.TryCopyTo(dest, out var written);
}
```