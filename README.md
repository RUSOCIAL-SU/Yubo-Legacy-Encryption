# Yubo-Legacy-Encryption

Old Yubo Encryption No Longer Used

```csharp
using SignerMiddleware.Models;
using System.Buffers.Binary;
using System.Security.Cryptography;
using System.Text;
using System.Text.Json;

public static class Xor
{
    public static byte[] Encrypt(ReadOnlySpan<byte> buffer)
    {
        const ulong keyConstant = 0x7DF927017F6DE25UL;

        // Prepare key.
        Span<byte> key = stackalloc byte[8];
        BinaryPrimitives.WriteUInt64LittleEndian(key, keyConstant);

        var output = new byte[buffer.Length];

        // Execute.
        for (var i = 0; i < buffer.Length; i++)
        {
            var a = i != 0 ? buffer[i - 1] : (byte)0;
            var b = (byte)(key[i & 7] ^ buffer[i]);
            var c = (byte)(b ^ a);

            output[buffer.Length - i - 1] = c;
        }

        return output;
    }

    public static byte[] Decrypt(ReadOnlySpan<byte> buffer)
    {
        const ulong keyConstant = 0x7DF927017F6DE25UL;

        // Prepare key.
        Span<byte> key = stackalloc byte[8];
        BinaryPrimitives.WriteUInt64LittleEndian(key, keyConstant);

        var output = new byte[buffer.Length];

        // Execute.
        for (var i = 0; i < buffer.Length; i++)
        {
            var a = i != 0 ? output[i - 1] : (byte)0;
            var b = (byte)(key[i & 7] ^ buffer[buffer.Length - i - 1]);
            var c = (byte)(b ^ a);

            output[i] = c;
        }

        return output;
    }
}
public class YuboFunctions
{

    public static string Signature(string path, SignatureJSON JSON)
    {
        using (HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String("IQnu686vmlRjK/4y2J5Rtw==")))
        {
            byte[] data = Encoding.UTF8.GetBytes(path + JsonSerializer.Serialize(JSON));
            byte[] hash = hmac.ComputeHash(data);
            return Convert.ToBase64String(hash);
        }
    }
    public static string SignatureLegacy(string username, string timestamp)
    {
        using (HMACSHA256 hmac = new HMACSHA256(Convert.FromBase64String("NwxVEdhbv2VGt8zRLlDeYw==")))
        {
            byte[] data = Encoding.UTF8.GetBytes(username + timestamp);
            byte[] hash = hmac.ComputeHash(data);
            return Convert.ToBase64String(hash);
        }
    }
    public static string GenerateWideVine()
    {
        int length = 16;
        const string chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        var randomBytes = new byte[length];

        using (var rng = RandomNumberGenerator.Create())
        {
            rng.GetBytes(randomBytes);
        }

        var randomString = new StringBuilder(length);
        foreach (var randomByte in randomBytes)
        {
            randomString.Append(chars[randomByte % chars.Length]);
        }

        return Convert.ToBase64String(Encoding.UTF8.GetBytes(randomString.ToString()));
    }

    public static int GenerateRandomUptime()
    {
        Random random = new Random();
        int minUptime = 0; // Minimum uptime value in seconds
        int maxUptime = 1000000; // Maximum uptime value in seconds

        int randomUptime = random.Next(minUptime, maxUptime);
        return randomUptime;
    }

    public static double GenerateRandomDouble(double min, double max)
    {
        var random = new Random();
        return random.NextDouble() * (max - min) + min;
    }

    public static long GenerateRandomUnixTimestamp()
    {
        Random random = new Random();
        long currentTimestampMillis = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds();
        long maxTimeDifferenceMillis = (long)TimeSpan.FromDays(10).TotalMilliseconds;

        long minTimestampMillis = currentTimestampMillis - maxTimeDifferenceMillis;
        long maxTimestampMillis = currentTimestampMillis;

        long randomTimestampMillis = (long)(random.NextDouble() * (maxTimestampMillis - minTimestampMillis) + minTimestampMillis);
        return randomTimestampMillis;
    }

    public static byte[] GenerateRandomIV()
    {
        using (Aes aes = Aes.Create())
        {
            aes.GenerateIV();
            byte[] iv = aes.IV;
            return iv;
        }
    }
}
```
