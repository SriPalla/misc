Here’s a guide to writing a **JUnit 5** test using **MockK** for the `uploadFile` function in Kotlin. The test will mock the `SftpConnectionFactory` and the `ChannelSftp` so that the SFTP operations can be tested in isolation without making actual connections.

### Key Steps:
1. **Mock `SftpConnectionFactory` and `ChannelSftp`**.
2. **Mock the behavior of `cd` (change directory) and `put` (upload file)**.
3. **Verify that the correct interactions occur**.
4. **Handle the `use` block behavior by ensuring the mock can be "closed" properly**.

### Code Implementation

```kotlin
import com.jcraft.jsch.ChannelSftp
import io.mockk.*
import org.junit.jupiter.api.Assertions.*
import org.junit.jupiter.api.Test
import java.io.File

class SftpUploadTest {

    // Mock SftpConnectionFactory and ChannelSftp
    private val sftpConnectionFactory = mockk<SftpConnectionFactory>(relaxed = true)
    private val channelSftp = mockk<ChannelSftp>(relaxed = true)

    @Test
    fun `should upload file to SFTP server with default remote directory`() {
        // Arrange: Set up the mocks
        val localFile = File("testfile.txt")
        val remoteFileName = "remoteFile.txt"
        val defaultRemoteDirectory = "/default-directory"

        // Mock the behavior of the factory and channel
        every { sftpConnectionFactory.getChannel() } returns channelSftp
        every { channelSftp.remoteDirectory } returns defaultRemoteDirectory

        // Act: Call the uploadFile function
        uploadFile(localFile, remoteFileName, null, sftpConnectionFactory)

        // Assert: Verify interactions with the mocks
        verify {
            sftpConnectionFactory.getChannel() // Channel should be retrieved
            channelSftp.cd(defaultRemoteDirectory) // Should change to the default remote directory
            channelSftp.put(localFile.absolutePath, remoteFileName) // Should upload the file
        }
        verify(exactly = 1) { sftpConnectionFactory.close() } // Verify that the factory is closed
    }

    @Test
    fun `should upload file to SFTP server with specified remote directory`() {
        // Arrange: Set up the mocks
        val localFile = File("testfile.txt")
        val remoteFileName = "remoteFile.txt"
        val specifiedRemoteDirectory = "/specified-directory"

        every { sftpConnectionFactory.getChannel() } returns channelSftp

        // Act: Call the uploadFile function with a specified directory
        uploadFile(localFile, remoteFileName, specifiedRemoteDirectory, sftpConnectionFactory)

        // Assert: Verify that the correct directory and upload calls are made
        verify {
            sftpConnectionFactory.getChannel()
            channelSftp.cd(specifiedRemoteDirectory) // Should use the specified directory
            channelSftp.put(localFile.absolutePath, remoteFileName) // Should upload the file
        }
        verify(exactly = 1) { sftpConnectionFactory.close() }
    }

    @Test
    fun `should throw runtime exception if file upload fails`() {
        // Arrange: Set up the mocks and make the upload fail
        val localFile = File("testfile.txt")
        val remoteFileName = "remoteFile.txt"
        val defaultRemoteDirectory = "/default-directory"

        every { sftpConnectionFactory.getChannel() } returns channelSftp
        every { channelSftp.remoteDirectory } returns defaultRemoteDirectory

        // Simulate an exception during the upload
        every { channelSftp.put(localFile.absolutePath, remoteFileName) } throws RuntimeException("SFTP failure")

        // Act & Assert: Expect a RuntimeException to be thrown
        val exception = assertThrows<RuntimeException> {
            uploadFile(localFile, remoteFileName, null, sftpConnectionFactory)
        }

        // Assert that the exception contains the correct message
        assertEquals("Failed to upload file to SFTP", exception.message)

        // Verify that the factory is still closed even if an exception occurs
        verify(exactly = 1) { sftpConnectionFactory.close() }
    }
}
```

### Explanation of Key Parts:

1. **Mock Setup**:
   - `mockk<SftpConnectionFactory>(relaxed = true)`: We mock the `SftpConnectionFactory` and the `ChannelSftp` objects. The `relaxed = true` ensures that any unstubbed method calls will return the default value for the type (like null for objects).
   - `every { sftpConnectionFactory.getChannel() } returns channelSftp`: This mocks the behavior where calling `getChannel()` on the `SftpConnectionFactory` returns the mocked `ChannelSftp`.

2. **Default Directory Handling**:
   - If `remoteDirectory` is not passed, we mock the default remote directory using `every { channelSftp.remoteDirectory } returns "/default-directory"`.

3. **Verifications**:
   - We use `verify` to ensure that the correct methods were called, e.g., `channelSftp.cd()` to change directories and `channelSftp.put()` to upload the file.
   - We also verify that the `use` block (i.e., autoclosable behavior) works by verifying that `sftpConnectionFactory.close()` is called exactly once.

4. **Exception Handling**:
   - In the third test, we simulate an exception using `throws RuntimeException("SFTP failure")` and verify that the exception is properly caught and rethrown with the appropriate message.

### Summary
- The test cases cover various scenarios such as uploading a file to the default or specified directory, and handling exceptions during the upload process.
- **MockK** is used to mock `SftpConnectionFactory` and `ChannelSftp` to isolate and test the `uploadFile` method without real SFTP connections.
