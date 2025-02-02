// MainActivity.kt
import android.os.Bundle
import android.util.Log
import android.view.WindowManager
import androidx.appcompat.app.AppCompatActivity
import com.google.mlkit.vision.common.InputImage
import com.google.mlkit.vision.face.Face
import com.google.mlkit.vision.face.FaceDetection
import com.google.mlkit.vision.face.FaceDetectorOptions

class MainActivity : AppCompatActivity() {
    private lateinit var detector: FaceDetector

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Initialize Face Detector with high accuracy settings
        val options = FaceDetectorOptions.Builder()
            .setPerformanceMode(FaceDetectorOptions.PERFORMANCE_MODE_ACCURATE)
            .setClassificationMode(FaceDetectorOptions.CLASSIFICATION_ALL)
            .build()

        detector = FaceDetection.getClient(options)

        // Start camera preview (you need to implement CameraX setup)
        startCamera()
    }

    private fun processImage(image: InputImage) {
        detector.process(image)
            .addOnSuccessListener { faces ->
                for (face in faces) {
                    checkEyeState(face)
                }
            }
            .addOnFailureListener { e ->
                Log.e("BlinkApp", "Face detection failed", e)
            }
    }

    private fun checkEyeState(face: Face) {
        // Threshold for considering eyes as closed
        val EYE_CLOSED_THRESHOLD = 0.1f

        if (face.leftEyeOpenProbability != null && 
            face.rightEyeOpenProbability != null &&
            face.leftEyeOpenProbability < EYE_CLOSED_THRESHOLD &&
            face.rightEyeOpenProbability < EYE_CLOSED_THRESHOLD) {
            
            turnOffScreen()
        }
    }

    private fun turnOffScreen() {
        // Method 1: Turn brightness to minimum
        val params = window.attributes
        params.screenBrightness = 0f
        window.attributes = params

        // Method 2: Lock the screen (requires device admin permission)
        // val deviceManager = getSystemService(DEVICE_POLICY_SERVICE) as DevicePolicyManager
        // deviceManager.lockNow()
    }

    // Implement CameraX setup here...
}
