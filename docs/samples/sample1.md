```c#

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
 
namespace HmacTest
{
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using System.Security.Cryptography;
    using System.Text;
    using System.Threading.Tasks;
 
    namespace Common.HttpClient
    {
 
        public interface IHttpRequestSigner
        {
            /// <summary>
            /// Method for signing requests.
            /// </summary>
            /// <param name="request">Request to sign.</param>
            Task SignRequestAsync(HttpRequestMessage request);
        }
 
        /// <summary>
        /// Hmac signing for http requests.
        /// </summary>
        public class HmacRequestSigner : IHttpRequestSigner, IDisposable
        {
            private readonly int _partnerId;
            private readonly HMAC _hmac;
            private readonly Func<DateTime> _utcDateTimeProvider;
 
            /// <summary>
            /// Creates a SHA-256 HmacRequestSigner from the given base-64 encoded secret key.
            /// </summary>
            /// <param name="partnerId">Common partner id.</param>
            /// <param name="base64EncodedSecretKey">Base64-encoded secret key.</param>
            /// <param name="utcDateTimeProvider">UTC datetime provider. DateTime.UtcNow will be used if null.</param>
            public static HmacRequestSigner CreateWithSha256(int partnerId, string base64EncodedSecretKey, Func<DateTime> utcDateTimeProvider = null)
            {
                return new HmacRequestSigner(partnerId, new HMACSHA256(Convert.FromBase64String(base64EncodedSecretKey)), utcDateTimeProvider);
            }
 
            /// <summary>
            /// Creates a new HmacRequestSigner.
            /// </summary>
            /// <param name="partnerId">Common partner id.</param>
            /// <param name="hmac">Hmac object to use.</param>
            /// <param name="utcDateTimeProvider">UTC datetime provider. DateTime.UtcNow will be used if null.</param>
            public HmacRequestSigner(int partnerId, HMAC hmac, Func<DateTime> utcDateTimeProvider = null)
            {
                if (hmac == null) throw new ArgumentNullException(nameof(hmac));
 
                _partnerId = partnerId;
                _hmac = hmac;
                _utcDateTimeProvider = utcDateTimeProvider ?? (() => DateTime.UtcNow);
            }
 
            /// <summary>
            /// Signs the request with a hmac authorization header.
            /// </summary>
            /// <param name="request">Request to sign.</param>
            public async Task SignRequestAsync(HttpRequestMessage request)
            {
                if (request == null) throw new ArgumentNullException(nameof(request));
 
                var requestContentMd5Hash = string.Empty;
                var requestUri = WebUtility.UrlEncode(request.RequestUri.AbsoluteUri.ToLower());
                var requestHttpMethod = request.Method.Method;
                var requestTimeStamp = Convert.ToInt64((_utcDateTimeProvider() - new DateTime(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc)).TotalSeconds);
                var nonce = Guid.NewGuid().ToString("N");
 
                if (request.Content != null)
                {
                    var content = await request.Content.ReadAsByteArrayAsync();
 
                    using (var md5 = MD5.Create())
                    {
                        requestContentMd5Hash = Convert.ToBase64String(md5.ComputeHash(content));
                    }
                }
 
                var signatureRawData = $"{_partnerId}{requestHttpMethod}{requestUri}{requestTimeStamp}{nonce}{requestContentMd5Hash}";
                var signature = Encoding.UTF8.GetBytes(signatureRawData);
 
                lock (_hmac)
                {
                    var hmac = Convert.ToBase64String(_hmac.ComputeHash(signature)).Substring(0, 10);
                    request.Headers.Authorization = new AuthenticationHeaderValue("hmac", $"\"{_partnerId}:{hmac}:{nonce}:{requestTimeStamp}\"");
                }
            }
 
            /// <summary>
            /// Performs application-defined tasks associated with freeing, releasing, or resetting unmanaged resources.
            /// </summary>
            public void Dispose()
            {
                _hmac?.Dispose();
            }
        }
    }
}

```