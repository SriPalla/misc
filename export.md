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
