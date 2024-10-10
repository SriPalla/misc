When you want to mock an object like `JSch` in Kotlin unit testing using MockK (or any other mocking library), but the object is **created inside the method** itself (e.g., `val jsch = JSch()`), it becomes a bit more complex. The reason is that the object creation is inside the method, and you cannot directly mock it unless you use advanced mocking techniques like **mocking constructors** or **dependency injection**.

Here are two approaches to handle this scenario:

### 1. Refactor to Use Dependency Injection
This is the cleanest approach. By injecting the `JSch` object (or `SftpConnectionFactory`, depending on how your code is structured), you can easily mock it in your unit tests.

#### Refactoring the Code

Instead of creating the `JSch` object inside the method, inject it through the constructor or method parameters.

```kotlin
class SftpUploader(private val jsch: JSch) {
    fun uploadFile(localFile: File, remoteFileName: String, remoteDirectory: String? = null, session: Session) {
        val remoteDir = remoteDirectory ?: "/default-directory"

        session.use { sftpSession ->
            val channelSftp = sftpSession.openChannel("sftp") as ChannelSftp
            channelSftp.connect()

            try {
                channelSftp.cd(remoteDir)
                channelSftp.put(localFile.absolutePath, remoteFileName)
            } catch (e: Exception) {
                throw RuntimeException("Failed to upload file to SFTP", e)
            } finally {
                channelSftp.disconnect()
            }
        }
    }
}
```

#### Unit Test with MockK

Now, in your unit test, you can easily mock the `JSch`, `Session`, and `ChannelSftp` objects:

```kotlin
import com.jcraft.jsch.*
import io.mockk.*
import org.junit.jupiter.api.Test
import java.io.File

class SftpUploaderTest {

    private val jsch = mockk<JSch>(relaxed = true)
    private val session = mockk<Session>(relaxed = true)
    private val channelSftp = mockk<ChannelSftp>(relaxed = true)

    @Test
    fun `should upload file to SFTP server`() {
        // Arrange
        val localFile = File("testfile.txt")
        val remoteFileName = "remoteFile.txt"
        val remoteDirectory = "/remote-directory"

        every { session.openChannel("sftp") } returns channelSftp
        every { channelSftp.cd(remoteDirectory) } just Runs
        every { channelSftp.put(localFile.absolutePath, remoteFileName) } just Runs

        // Act
        SftpUploader(jsch).uploadFile(localFile, remoteFileName, remoteDirectory, session)

        // Assert
        verify {
            session.openChannel("sftp")
            channelSftp.cd(remoteDirectory)
            channelSftp.put(localFile.absolutePath, remoteFileName)
        }
    }
}
```

### 2. Mocking Object Creation (Constructor) Inside Method
If refactoring is not an option (for example, due to constraints in your project or legacy code), you can use **MockK's `mockkConstructor`** feature to mock the construction of objects like `JSch` inside your method.

#### Example without Refactoring

Here is an example where the `JSch` object is still created inside the method, but we mock it:

```kotlin
fun uploadFile(localFile: File, remoteFileName: String, remoteDirectory: String? = null) {
    val jsch = JSch() // This will be mocked
    val session = jsch.getSession("username", "host", 22)

    session.use { sftpSession ->
        val channelSftp = sftpSession.openChannel("sftp") as ChannelSftp
        channelSftp.connect()

        try {
            val remoteDir = remoteDirectory ?: "/default-directory"
            channelSftp.cd(remoteDir)
            channelSftp.put(localFile.absolutePath, remoteFileName)
        } catch (e: Exception) {
            throw RuntimeException("Failed to upload file to SFTP", e)
        } finally {
            channelSftp.disconnect()
        }
    }
}
```

#### Unit Test with MockK's `mockkConstructor`

In this case, we use `mockkConstructor` to mock the `JSch` and `Session` instances created inside the method.

```kotlin
import com.jcraft.jsch.*
import io.mockk.*
import org.junit.jupiter.api.Test
import java.io.File

class SftpUploaderTest {

    @Test
    fun `should upload file to SFTP server when JSch is created inside the method`() {
        // Arrange
        val localFile = File("testfile.txt")
        val remoteFileName = "remoteFile.txt"
        val remoteDirectory = "/remote-directory"

        // Mock JSch, Session, and ChannelSftp
        mockkConstructor(JSch::class)
        val session = mockk<Session>(relaxed = true)
        val channelSftp = mockk<ChannelSftp>(relaxed = true)

        every { anyConstructed<JSch>().getSession(any(), any(), any()) } returns session
        every { session.openChannel("sftp") } returns channelSftp
        every { channelSftp.cd(remoteDirectory) } just Runs
        every { channelSftp.put(localFile.absolutePath, remoteFileName) } just Runs

        // Act
        uploadFile(localFile, remoteFileName, remoteDirectory)

        // Assert
        verify {
            anyConstructed<JSch>().getSession(any(), any(), any())
            session.openChannel("sftp")
            channelSftp.cd(remoteDirectory)
            channelSftp.put(localFile.absolutePath, remoteFileName)
        }
    }
}
```

### Key Points:
1. **`mockkConstructor(JSch::class)`** allows you to mock the constructor of the `JSch` class when it is instantiated inside the method.
2. **`anyConstructed<JSch>()`** is used to mock interactions with any `JSch` object constructed inside the method.
3. We mock the `Session`, `ChannelSftp`, and their methods (e.g., `cd`, `put`) to simulate the SFTP behavior without making real connections.

### Summary:
- **Refactor** the code to inject dependencies like `JSch` and `Session` to make testing easier.
- If refactoring is not possible, use **`mockkConstructor`** to mock object creation inside the method.
