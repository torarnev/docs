
```

/*
* Copyright (c) 2017 LINK Mobility Oy. All rights reserved.
*/
 
package com.labyrintti.silverbullet.payment;
 
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.security.InvalidKeyException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Date;
import java.util.UUID;
 
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
 
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.lang3.StringUtils;
 
/**
* <p>
* Authorization header for the HTTP requests to the LINK Payment API. Such header contains:
* <ul>
* <li>payment credentials identifying a Payment servie's partner</li>
* <li>information about the request: HTTP method, URL and payload(body)</li>,
* <li>unique identifiers for the header: timestamp and UUID</li>
* </ul>
* .
* </p>
*
*/
public class AuthorizationHeader {
 
private static final String HEADER_NAME = "Authorization";
/**
* The encoding used for the header
*/
private static final String ENCODING = "UTF-8";
 
/**
* Identifier for the MD5 algorithm as expected from {@link MessageDigest java.security.MessageDigest}
*/
private static final String MD5_ALGORITHM_NAME = "MD5";
 
/**
* Identifier for the HmacSHA256 algorithm as expected from {@link javax.crypto.Mac}
*/
private static final String HMAC_SHA256_ALGORITHM_NAME = "HmacSHA256";
 
/**
* The number of characters from the signature used in the header
*/
private static final int HEADER_SIGNATURE_LENGTH = 10;
 
/**
* The symbol used to separate the parts of the header's value
*/
private static final String HEADER_PARTS_SEPARATOR = ":";
 
/**
* A prefix used for the header's value
*/
private static final String HEADER_PREFIX = "hmac ";
 
/**
* A partner ID used to identify the partner
*/
private Integer partnerId;
 
/**
* An authentication secret key for the partner
*/
public String secretKey;
 
/**
* UNIX time (the number of seconds since Epoch UTC (January 01, 1970)) used to identify a request
*/
private long timestampInSeconds;
 
/**
* A UUID used to identify a request
*/
private UUID uuid;
 
/**
* The HTTP method of the request
*/
private String httpMethod;
 
/**
* The absolute URL of the request including any query parameters
*/
private String url;
 
/**
* The payload(body) of the request
*/
private String requestPayload;
 
/**
* Constructs AuthorizationHeader with the provided data and internally generated identifiers - timestamp (the
* current date) and UUID
*
* @param partnerId the id of the Payment service's partner
* @param secretKey the secret key used for authentication
* @param httpMethod the HTTP method of the request
* @param url the absolute URL of the request including any query parameters
* @param requestPayload the payload(body) of the request; the payload is not a required value and both null and
* empty string are accepted
*/
public AuthorizationHeader(Integer partnerId, String secretKey, String httpMethod, String url, String requestPayload) {
super();
this.partnerId = partnerId;
this.secretKey = secretKey;
this.httpMethod = httpMethod;
this.url = url;
this.requestPayload = requestPayload;
 
initHeaderIdentifiers();
}
 
/**
* Returns the authorization header value for the Payment API and returns it. The header value should be in follow
* the pattern: "hmac {partnerId}:{signature}:{nonce}:{timestamp}" where:
* <ul>
* <li>{partner} the partner id</li>
* <li>{signature} the first 10 characters of a Hmac Sha256 hash used as signature</li>
* <li>{nonce} a UUID that is unique for each header</li>
* <li>{timestamp} the partner id</li>
*
* @return the value of the authorization header
* @throws AuthorizationHeaderValueConstructionException in case the header value cannot be constructed
*/
public String getHeaderValue() throws AuthorizationHeaderValueConstructionException {
String headerValue = null;
try {
headerValue = constructHeaderValueFromSourceData();
} catch (InvalidKeyException | UnsupportedEncodingException | NoSuchAlgorithmException e) {
throw new AuthorizationHeaderValueConstructionException(e);
}
return headerValue;
}
 
/**
* Returns the name of the authorization header's name
*
* @return the name of the authorization header
*/
public String getHeaderName() {
return HEADER_NAME;
}
 
/**
* Constructs the authorization header value for the Payment API from the header's source data
*
* @return the value of the authorization header
* @throws UnsupportedEncodingException in case the {@link #ENCODING header's encoding} is not supported
* @throws NoSuchAlgorithmException in case any of the hashing algorithms used is not supported
* @throws InvalidKeyException in case the privateSecretKey is inappropriate
*/
private String constructHeaderValueFromSourceData() throws UnsupportedEncodingException, NoSuchAlgorithmException, InvalidKeyException {
String message = generateMessageToBeSigned();
String signature = sign(message);
String header = buildAuthorizationHeaderString(signature);
 
return header;
}
 
/**
* Initiates the values of the values that identify a concrete header: it's timestamp and UUID
*/
private void initHeaderIdentifiers() {
this.timestampInSeconds = new Date().getTime() / 1000;
this.uuid = UUID.randomUUID();
}
 
/**
* Generates a string (called "message") used to create signature for the header. This message is constructed from
* the request data, the {@link #partnerId partner ID}, the {@link #timestampInSeconds} and the {@link #uuid}
*
* @return a message used to create a header's signature
* @throws UnsupportedEncodingException in case the {@link #ENCODING} is not supported by the URL encoder or
* string-to-byte encoder
* @throws NoSuchAlgorithmException in case the hashing algorithm is not supported
*/
private String generateMessageToBeSigned() throws UnsupportedEncodingException, NoSuchAlgorithmException {
String upperCaseHttpMethodName = httpMethod.toUpperCase();
String timestampAsString = Long.toString(timestampInSeconds);
String lowerCaseUrl = url.toLowerCase();
String urlEncodedLowerCaseUrl = URLEncoder.encode(lowerCaseUrl, ENCODING);
String requestPayloadEncodedHash = StringUtils.EMPTY;
if (requestPayload != null && !StringUtils.isEmpty(requestPayload)) {
requestPayloadEncodedHash = getHashedAndEncodedRequestPayload();
}
 
String messageToBeSigned = partnerId + upperCaseHttpMethodName + urlEncodedLowerCaseUrl + timestampAsString + uuid + requestPayloadEncodedHash;
 
return messageToBeSigned;
}
 
/**
* Creates a HMAC signature string for the header
*
* @param message an input message to be signed
* @return the provided message hashed with HMAC-SHA256 using the {@link #privateSecretKey}
* @throws UnsupportedEncodingException in case the charset used to encode the message into a byte array
* @throws InvalidKeyException in case the {@link #privateSecretKey} is inappropriate for MAC
* @throws NoSuchAlgorithmException in case the selected MAC algorithm (HMAC-SHA256) is not supported
*/
private String sign(String message) throws UnsupportedEncodingException, InvalidKeyException, NoSuchAlgorithmException {
byte[] decodedSecretKeyByteArray = getDecodedSecretKeyAsByteArray();
byte[] messageByteArray = message.getBytes(ENCODING);
 
byte[] hash = createSha256Hash(decodedSecretKeyByteArray, messageByteArray);
String encodedHash = Base64.encodeBase64String(hash);
 
return encodedHash;
}
 
/**
* Builds a authorization header value using part of the provided signature by joining all the header parts (the
* {@link #partnerId partner ID}, the first characters of the signature, the {@link #uuid} and the
* {@link #timestampInSeconds}) separated by {@link AuthorizationHeader#HEADER_PARTS_SEPARATOR} and adding a
* {@link #HEADER_PREFIX prefix} to the string
*
* @param fullSignature the full signature for the header
* @return an authorization header that uses the provided signature
*/
private String buildAuthorizationHeaderString(String fullSignature) {
String timestampAsString = Long.toString(timestampInSeconds);
String headerSignature = fullSignature.substring(0, HEADER_SIGNATURE_LENGTH);
String partnerIdAsString = String.valueOf(partnerId);
String uuidString = uuid.toString();
 
String joinedHeaderParts = joinStringsWithSeparator(HEADER_PARTS_SEPARATOR, partnerIdAsString, headerSignature, uuidString, timestampAsString);
String headerValue = HEADER_PREFIX + joinedHeaderParts;
return headerValue;
}
 
/**
* Hashes and encodes the request payload (body) using MD5 and Base64 respectively
*
* @return the hashed and then encoded payload
* @throws NoSuchAlgorithmException in case the hash algorithm is not supported
* @throws UnsupportedEncodingException in case the encoding used for th header is not supported
*/
private String getHashedAndEncodedRequestPayload() throws NoSuchAlgorithmException, UnsupportedEncodingException {
byte[] requestPayloadHash = createMd5Hash(requestPayload);
String requestPayloadEncodedHash = Base64.encodeBase64String(requestPayloadHash);
 
return requestPayloadEncodedHash;
}
 
/**
* Returns the secret key decoded with Base64
*
* @return the decoded secret key
*/
private byte[] getDecodedSecretKeyAsByteArray() {
byte[] decodedSecretKey = Base64.decodeBase64(secretKey);
 
return decodedSecretKey;
}
 
/**
* Creates a MD5 hash from the provided input
*
* @param input the input string to hash
* @return the creates hash as binary data
* @throws NoSuchAlgorithmException in case the selected algorithm (MD4) is not supported
* @throws UnsupportedEncodingException in case the encoding of the input character set is not supported
*/
private static byte[] createMd5Hash(String input) throws NoSuchAlgorithmException, UnsupportedEncodingException {
MessageDigest md5Digest = MessageDigest.getInstance(MD5_ALGORITHM_NAME);
md5Digest.update(input.getBytes(ENCODING));
byte[] hash = md5Digest.digest();
 
return hash;
}
 
/**
* Creates HmacSHA256 hash of a message using the provided secret key
*
* @param secretKey a secret key used to create the hash
* @param message the message to be hashed and authenticated/validated later
* @return the created hash as binary data
* @throws NoSuchAlgorithmException in case the selected algorithm (HmacSHA256) is not supported
* @throws InvalidKeyException in case the provided key is inappropriate for the MAC
*/
private static byte[] createSha256Hash(byte[] secretKey, byte[] message) throws NoSuchAlgorithmException, InvalidKeyException {
Mac algorithm = Mac.getInstance(HMAC_SHA256_ALGORITHM_NAME);
SecretKeySpec secretKeySpec = new SecretKeySpec(secretKey, HMAC_SHA256_ALGORITHM_NAME);
algorithm.init(secretKeySpec);
byte[] hash = algorithm.doFinal(message);
 
return hash;
}
 
/**
* <p>
* Joins the elements of the provided array into a single String containing the provided list of elements.
* </p>
*
* @see StringUtils#join(Object[], String)
* @param separator the separator to use, null treated as ""
* @param stringsToJoin the varargs strings providing the values to join together. null elements are treated as ""
* @return the joined String, {@code null} if no strings to join are provided
*/
private static String joinStringsWithSeparator(String separator, String... stringsToJoin) {
String joined = StringUtils.join(stringsToJoin, separator);
return joined;
}
 
}

```