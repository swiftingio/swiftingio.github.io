---
layout: post
author: bartek
title: \#33 Security – implement your own encryption schema
excerpt: 
---
After publishing this post and shared it via Twitter, on the next day we got a message from [Rob Napier](https://twitter.com/cocoaphony?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post):

![](https://github.com/swiftingio/blog/raw/%2333-Master-Key-encryption/Images/twitter.png/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Next I have contacted Rob, who helped me a lot with improving this post by pointing out main issues in the article and in the example project. I have to say that Rob helped me a lot with understanding security concepts by his feedback and his great knowledge. Big thanks to Rob Napier for that!!!

The article below contains improvements based on Rob's comments. Updated sections are marked with red exclamation marks: **❗Updated❗**.

#### Introduction

I hope you you've had possibility to look at our [previous post](https://swifting.io/blog/2016/08/09/21-ios-security-101/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) about security basics. 

Today we would like to concentrate more on scenarios in which we want to strictly secure our sensitive data in our applications, with below assumptions:

- Users can have insecure or default passcode,
- Users have devices without a passcode.

In these cases it is good to consider implementation of own encryption scheme.

**Note:** You can find example project, based on snippets shown below, on my GitHub [account](https://github.com/barcis89/masterKeyEncryption/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Basics

Apple provides many different aspects of security. During development of your security layer don't forget about crucial things like:

- **Keychain** - A lot of apps handle passwords and other short sensitive bits of data like: keys, login, passwords, tokens. The iOS Keychain provides a secure way to store these items. Under the hood, the Keychain is simply a SQLite database. Keychain data is protected using a [class](https://developer.apple.com/reference/security/1658642-keychain_services/1663541-keychain_item_accessibility_cons/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) structure, which determine access level to stored data. 
- **Data Protection** - Every time we create a file in our iOS application we can assign a specific [class](https://developer.apple.com/reference/foundation/nsfilemanager/1653059-file_protection_values/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)) which determines the access level to the created resource.
- **Passcode** - Without a passcode, the entire content of the Keychain can be read by anyone with physical access to the device. The stronger it is, the stronger the encryption key becomes. Brute-forcing 4-digit PINs would take 4-5 days (according to [this](https://www.consumeraffairs.com/news/how-to-brute-force-break-a-4-digit-iphone-password-in-111-hours-or-less-032515.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) source), but a 6-character alphanumeric passcode with lowercase letters, uppercase letters and numbers would take years to break it down.

#### Implementing an encryption scheme

What does it mean to implement an encryption scheme? It means that we need to:

- choose encryption algorithm (**AES** is commonly used and recommended)
- generate a derived key using **PBKDF** algorithm, which can be used as the encryption key to encrypt our data.

Is that all? Unfortunately not yet...

What would it mean if user wanted to change his password? It would mean that we had to decrypt and re-encrypt all of the user's data if the password was tied directly to the encrypted data. Fortunately a better solution, which is based on using a master key concept, exists:

![](https://raw.githubusercontent.com/swiftingio/blog/%2333-Master-Key-encryption/Images/MasterKeyEncryprion.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

A master key never needs to change if it is protected at all times. When user wants to change his password, then the master key can simply be re-encrypted with the new derived key. Below schema shows process of changing their password in perspective of decrypting and encrypting the master key:

![](https://raw.githubusercontent.com/swiftingio/blog/%2333-Master-Key-encryption/Images/MasterKey.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Finally, having the decrypted master key we can encrypt our data with an AES algorithm:

![](https://raw.githubusercontent.com/swiftingio/blog/%2333-Master-Key-encryption/Images/DataEncyptionIV.jpg?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)

Ok, but what are Salt, PBKDF, AES, IV, HMAC, etc.? 

###### User input - password, pin

To create our own encryption, first thing we need is some user password or pin, which is used as an input to calculate a derived key. Using the password/pin as the only input to derive your encryption key can be brute-forced quite quickly. That's why it is good to add some salt to calcuations.

###### Salt

Salt is just a random data. It is used as an additional input to a one-way function that "hashes" a password or pin. It is used to make brute-force and [rainbow table](https://en.wikipedia.org/wiki/Rainbow_table/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) attacks against your implementation computationally expensive.

Below example implementation of generating salt is based on proposal from [The Mobile Application Hacker's Handbook](http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118958500.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post):

```
    func generateSalt(length: Int) -> Data? {
        
        var data = Data(count: length)
        let result = data.withUnsafeMutableBytes { mutableBytes in
            SecRandomCopyBytes(kSecRandomDefault, data.count, mutableBytes)
        }
        
        if(result != 0) {
            print("Unable to generate salt")
            return nil
        }
        
        return data
    }
```

[Here](https://crackstation.net/hashing-security.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) you can find more about the concept of salt.

###### PBKDF2 (Password-Based Key Derivation Function 2)

PBKDF2 applies a pseudorandom function, such as a cryptographic hash, cipher, or hash-based message authentication code (HMAC), to the user input password/pin along with a salt value and repeats the process many times to produce a derived key, which can then be used as a cryptographic key in subsequent operations. The added computational work makes password cracking much more difficult, and is known as key stretching.

```
DK = PBKDF2(PRF, Password, Salt, c, dkLen)
```

- `PRF` is a pseudorandom two-argument function with output length `hLen` (e.g. a keyed [HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post))
- `c`: a desired number of iterations
- `dkLen`: a desired length of the derived key
- `DK`: a generated derived key

###### Derived Key

The Derived Key is an output of PBKDF2 function. In the concept shown on the pictures it is used as the encryption key in AES algorithm to encrypt the master key.

###### AES and IV **❗Updated❗** 

The Advanced Encryption Standard (AES) is a symmetric encryption algorithm that supports a block length of 128 bits and key lengths of 128, 192, and 256 bits. AES in most libraries use [CBC](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post#Cipher_Block_Chaining_.28CBC.29) mode by default, that is used generally in database applications.

Initialization Vector (IV) is an unpredictable random number used to make sure that when the same message is encrypted twice, the cipher text is always different. It should be generated as a random data and stored with the cipher text.

It is very important not to forget to add the IV when calling AES function because it significantly increases our security schema. Generally the IV parameter is optional by default in many crypto libraries.

```
let iv: [UInt8] = AES.randomIV(AES.blockSize)
let aes = try AES(key: derivedKey, iv: iv, blockMode: .CBC, padding: PKCS7())
```

If you would like to explore AES algorithm more I strongly recommend watching [this video](https://www.youtube.com/watch?v=NHuibtoL_qk&t=784s&utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

###### Master key **❗Updated❗** 🔑

The Master Key is usually a randomly generated sequence of bits, which is used as a key to encrypt and decrypt application data by using AES algorithm. The longer it is, the longer the duration of encryption process is. It can really impact your application performance!

###### HMAC **❗Updated❗** 

HMAC is a hash-based message authentication code. It is a mechanism for calculating a message authentication code involving a hash function in combination with a secret key. Any general advice on encryption should always recommend authentication. Without authentication it can be possible to modify the cipher text (without knowing the key). The most common way to do this is to generate a second key and use it to generate the HMAC of the cipher text and append that to the end of the message.
 
#### Questions

When implementing your security layer for sure you will have a lot of questions. For me they were:

***1) Is data stored in Keychain with `kSecAttrAccessibleAlways` flag also protected?***

This class key is protected only with the [UID (Unique ID)](https://www.apple.com/business/docs/iOS_Security_Guide.pdf), and is kept in [Effaceable Storage](https://www.apple.com/business/docs/iOS_Security_Guide.pdf). 

***2) Where to store the Master key?***

The encrypted Master Key can be stored for example in Keychain.

***3) How long should be the Salt? ❗Updated❗***

It depends. In some cases 16 is plenty and in other cases 8 is plenty. And of course remember about decreased app performance when you use a long salt value 😀.

***4) What is the purpose of `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly` Keychain class?❗Updated❗***

If the user removes their passcode, all the Keychain items with this access restriction will be removed. Use it when your application requires using a passcode. It also prevents the item from being backed up to iCloud or moved to another device.

#### Remember
- Never implement your own crypto, unless you have to! There a few great Crypto libraries on GitHub. 
- Use at minimum 256-bit AES for symmetric key encryption.
- Use SHA-256 or SHA-512 as the hashing algorithm.
- Never reuse the salt.
- Use an unpredictable Initialization Vector. 
- Use AES with CBC mode.
- Keychain’s contents are not secure on a jailbroken device.
- Use an well-known key derivation function, such as `PBKDF2`, to generate keys.
- An attacker in possession of a device can’t get access to data with specific protection classes, if he doesn't know the passcode. Using Keychain is strongly recommended if you want to store sensitive data!
- If you create a new file, set its access level with Data Protection flags.
- Never hardcode your passwords in code - attackers can decompile your code!
- And last but not least use frameworks or libraries which can do security stuff for you.

#### Libraries

- [CryptoSwift](https://github.com/krzyzanowskim/CryptoSwift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) is a growing collection of standard and secure cryptographic algorithms implemented in Swift (AES, Hash functions, PBKDF functions and much more!). Made by [Marcin Krzyżanowski](http://www.krzyzanowskim.com/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).
- [keychain-swift](https://github.com/marketplacer/keychain-swift/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) contains helper functions for saving text in keychain securely for iOS, OS X, tvOS and watchOS.
- [RNCryptor](https://github.com/RNCryptor/RNCryptor?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) Cross-language AES Encryptor/Decryptor data format. Great library supporting a lot of security issues. I strongly recommend to use it if you would like to apply mechanisms mentioned in this post. Made by [Rob Napier](http://robnapier.net?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post).

#### Resources

- [Our example project](https://github.com/barcis89/masterKeyEncryption?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post) 
- [How to store salt](http://security.stackexchange.com/questions/17421/how-to-store-salt/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Hashing security](https://crackstation.net/hashing-security.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [The Mobile Application Hacker's Handbook](http://eu.wiley.com/WileyCDA/WileyTitle/productCd-1118958500.html?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Hacking and Securing iOS Applications](http://shop.oreilly.com/product/0636920023234.do?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [iOS Security Guide](https://www.apple.com/business/docs/iOS_Security_Guide.pdf)
- [Protecting the iOS Keychain](http://spr.com/ios-security-protecting-the-ios-keychain/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
- [Salt icon designed by Freepik](http://www.flaticon.com/free-icon/salt_12066/?utm_source=swifting.io&utm_medium=web&utm_campaign=blog%20post)
