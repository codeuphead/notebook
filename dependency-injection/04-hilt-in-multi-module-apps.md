# Hilt in multi-module apps

```kotlin
internal class MyLocaltionListener(
        private val context: Context,
        private val callback: (Location) -> Unit
) {
    fun start() {

    }

    fun stop() {

    }
}



class MyActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener

    override fun onCreate(...) {
        myLocationListener = MyLocationListener(this) {
            location ->
        }
    }

    public override fun onStart() {
        super.onStart()
        myLocationListener.start()
    }

    public override fun onStop(){
        super.onStop()
        myLocationListener.stop()
    }
}
```
