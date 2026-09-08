---
title: Client-Side Session Management - Storing and Protecting Tokens on Android and iOS
author: Ramzi Eljabali
date: 2026-08-08
tags:
  - article
  - session
  - session-managment
---
## Key Points

1. **What is a session**
2. **What makes a session**
3. **How to manage a session, client-side**
4. **The Architecture**
	1. **Shared: DataStore + pluggable encryption**
	2. **Android: DataStore + Keystore-backed encryption**
	3. **iOS: DataStore + Keychain-stored key**
5. **Final thoughts**
## 1. What is a session?

A session is a period of time during which a user is verified and known to the system they're using. That period grants the user access to processes and tools that otherwise wouldn't be available - without it, every request would need to re-prove identity from scratch.

From the client app's perspective, "managing a session" really means: **holding onto proof of identity, keeping it safe while it sits on the device, and knowing when and how to refresh it.**

## 2. What makes a session?

A session is comprised of two things:

**Access Token**
- Short-lived (minutes, typically)
- Sent with every request to prove the user is authenticated
- Doesn't need to be stored carefully long-term, since it expires quickly anyway

**Refresh Token**
- Long-lived (days or weeks)
- Used to obtain a new access token once the current one expires
- Also has its own expiry - if the user doesn't interact with the app within that window, the refresh token expires too, and they'll need to log in again

The refresh token is the more valuable of the two from a security standpoint. An access token that leaks is only useful for a few minutes. A refresh token that leaks can be used to mint new access tokens indefinitely, until it's rotated, revoked, or expires - which is why it deserves the most protection on the client.

## 3. How to manage a session, client-side

At a high level, the app's job is:

1. **Store** the access token in memory only - a plain variable, not persisted to disk. It's short-lived and gets rebuilt on every app launch anyway, so there's no reason to expose it to disk at all.
2. **Persist** the refresh token to secure, encrypted storage, since it needs to survive app restarts.
3. **Attach** the access token to outgoing requests.
4. **Detect** expiry (a 401 response, or a locally-decoded JWT `exp` claim) and trigger a refresh using the refresh token.
5. **Rotate** the refresh token on every use - get a new refresh token back with each refresh response, and discard the old one. If a discarded refresh token is ever presented again, that's a signal it may have been stolen, and the session should be revoked.
6. **Clear everything** on logout - wipe both tokens from memory and from secure storage.

The one part of this that's genuinely platform-specific is step 2 - how "secure, encrypted storage" actually works differs meaningfully between Android and iOS.

## 4. The architecture

I want to build this out in a way where both Android and iOS just need to implement the cryptography interface. Then we can simply inject them into the common `DataStoreSessionStorage`, to be used to decrypt and encrypt the keys to a common storage location defined by each platform.

- `CryptographyManager` 

```kotlin
interface CryptographyManager {  
    fun encrypt(value: String): String  
    fun decrypt(value: String): String  
}
```

- `SessionStorage`

```kotlin
interface SessionStorage {  
    fun observeAuthInfo(): Flow<AuthInfo?>  
  
    suspend fun setAuthInfo(authInfo: AuthInfo?)  
}
```

### Shared: DataStore + pluggable encryption

```kotlin
fun createDataStore(producePath: () -> String): DataStore<Preferences> = PreferenceDataStoreFactory.createWithPath {  
    producePath().toPath()  
}  
  
internal const val DATA_STORE_FILE_NAME = "prefs.preferences_pb"
```

```kotlin
class DataStoreSessionStorage(  
    private val dataStore: DataStore<Preferences>,  
    private val cryptographyManager: CryptographyManager  
) : SessionStorage {  
    private val authInfoKey = stringPreferencesKey("KEY_AUTH_INFO")  
    private val json =  
        Json {  
            ignoreUnknownKeys = true  
        }  
  
    override fun observeAuthInfo(): Flow<AuthInfo?> = dataStore.data.map { preferences ->  
        val serializedJson = preferences[authInfoKey]  
        serializedJson?.let {  
            try {  
                val decryptedAuthInfo = cryptographyManager.decrypt(it)  
                json.decodeFromString<AuthInfoSerializable>(decryptedAuthInfo).toDomain()  
            } catch (e: Exception) {  
                // Corrupted, tampered, or undecryptable session data - treat as logged out  
                null  
            }  
        }  
    }  
    override suspend fun setAuthInfo(authInfo: AuthInfo?) {  
        if (authInfo == null) {  
            dataStore.edit {  
                it.remove(authInfoKey)  
            }  
            return  
        }  
        val serializedJson = json.encodeToString(authInfo.toSerializable())  
        val encryptedString = cryptographyManager.encrypt(serializedJson)  
        dataStore.edit { preferences ->  
            preferences[authInfoKey] = encryptedString  
        }  
    }  
}
```


### Android: DataStore + Keystore-backed encryption

`DataStore<Preferences>` is Google's current recommended replacement for `SharedPreferences`, but it's important to understand what it actually provides: DataStore itself has no encryption at all. It's just an async, type-safe key-value store. If you write a refresh token into it directly, it sits in a plaintext file on the device's disk - readable on a rooted device, or extractable via a misconfigured backup.

The current recommended approach is **DataStore + Google Tink**, where:
- DataStore handles the storage.
- Tink handles the encryption, using a key that's generated and held in the Android Keystore - hardware-backed, meaning the raw key material never leaves secure hardware and can't be extracted even with root access.

Conceptually:

```kotlin
fun createDataStore(context: Context): DataStore<Preferences> = createDataStore {  
    context.filesDir.resolve(DATA_STORE_FILE_NAME).absolutePath  
}
```

```kotlin
class AeadCyptography(  
    context: Context,  
) : CryptographyManager {  
    companion object {  
        private const val KEY_SET_NAME = "keyset"  
        private const val PREFS_FILE_NAME = "keyset_prefs"     
		private const val MASTER_KEY_URI = "android-keystore://master_key"     
	}  
      
    init { 
	    AeadConfig.register()  
	}  
    
    private val aead =  
        AndroidKeysetManager.Builder()  
            .withSharedPref(context, KEY_SET_NAME, PREFS_FILE_NAME)  
	            .withKeyTemplate(KeyTemplate.createFrom(PredefinedAeadParameters.AES256_GCM))  
            .withMasterKeyUri(MASTER_KEY_URI)  
            .build()  
            .keysetHandle  
            .getPrimitive(  
                RegistryConfiguration.get(),  
                Aead::class.java,  
            )  
  
    override fun encrypt(value: String): String {  
        val ciphertext = aead.encrypt(  
            value.toByteArray(),  
            null,  
        )  
  
        return Base64.encodeToString(  
            ciphertext,  
            Base64.NO_WRAP,  
        )  
    }  
  
    override fun decrypt(value: String): String {  
        val ciphertext = Base64.decode(  
            value,  
            Base64.NO_WRAP,  
        )  
  
        return aead.decrypt(  
            ciphertext,  
            null,  
        ).decodeToString()  
    }  
}
```


### iOS: DataStore + Keychain-stored key

```kotlin
@OptIn(ExperimentalForeignApi::class)  
fun createDataStore(): DataStore<Preferences> = createDataStore {  
    val directory =  
        NSFileManager.defaultManager.URLForDirectory(  
            directory = NSDocumentDirectory,  
            inDomain = NSUserDomainMask,  
            appropriateForURL = null,  
            create = false,  
            error = null,  
        )  
    requireNotNull(directory).path + "/$DATA_STORE_FILE_NAME"  
}
```

```kotlin
@OptIn(ExperimentalForeignApi::class, CryptographyProviderApi::class)  
class NativeCryptography : CryptographyManager {  
  
    private val service = "ramzi.eljabali.justmessage.session"  
    private val keyAlias = "session_encryption_key"  
  
    private val provider = CryptographyProvider.Default  
    private val aesGcm = provider.get(AES.GCM)  
  
    private val key by lazy {  
        readKeyFromKeychain()?.let {  
            aesGcm.keyDecoder().decodeFromByteArrayBlocking(AES.Key.Format.RAW, it)  
        } ?: aesGcm.keyGenerator(keySize = AES.Key.Size.B256).generateKeyBlocking().also {  
            writeKeyToKeychain(it.encodeToByteArrayBlocking(AES.Key.Format.RAW))  
        }  
    }  
    @OptIn(ExperimentalEncodingApi::class)  
    override fun encrypt(value: String): String {  
        val ciphertext = key.cipher().encryptBlocking(value.encodeToByteArray())  
        return Base64.encode(ciphertext)  
    }  
  
    @OptIn(ExperimentalEncodingApi::class)  
    override fun decrypt(value: String): String {  
        val plaintext = key.cipher().decryptBlocking(Base64.decode(value))  
        return plaintext.decodeToString()  
    }  
  
    private fun baseQuery(): CFMutableDictionaryRef? {  
        val dict = CFDictionaryCreateMutable(  
            null,  
            3,  
            kCFTypeDictionaryKeyCallBacks.ptr,  
            kCFTypeDictionaryValueCallBacks.ptr,  
        )  
        CFDictionaryAddValue(dict, kSecClass, kSecClassGenericPassword)  
        CFDictionaryAddValue(dict, kSecAttrService, CFBridgingRetain(service))  
        CFDictionaryAddValue(dict, kSecAttrAccount, CFBridgingRetain(keyAlias))  
        return dict  
    }  
  
    private fun writeKeyToKeychain(bytes: ByteArray) {  
        val deleteQuery = baseQuery()  
        SecItemDelete(deleteQuery)  
  
        val addQuery = baseQuery()  
        CFDictionaryAddValue(addQuery, kSecValueData, CFBridgingRetain(bytes.toNSData()))  
        CFDictionaryAddValue(addQuery, kSecAttrAccessible, kSecAttrAccessibleWhenUnlockedThisDeviceOnly)  
        SecItemAdd(addQuery, null)  
    }  
  
    private fun readKeyFromKeychain(): ByteArray? {  
        val query = baseQuery()  
        CFDictionaryAddValue(query, kSecReturnData, CFBridgingRetain(true))  
        CFDictionaryAddValue(query, kSecMatchLimit, kSecMatchLimitOne)  
  
        memScoped {  
            val result = alloc<CFTypeRefVar>()  
            val status = SecItemCopyMatching(query, result.ptr)  
            if (status != errSecSuccess) return null  
            val data = CFBridgingRelease(result.value) as? NSData ?: return null  
            return data.toByteArray()  
        }  
    }  
}
```

## 5. Final thoughts

iOS takes a similar shape. The underlying goal is identical on both platforms - protect the refresh token at rest, since it's the most valuable thing your app holds. But how much of that protection the OS gives you for free is different, and that's the thing worth remembering. on Android you have to reach for encryption explicitly, on iOS you have to remember it persists past uninstall. 
