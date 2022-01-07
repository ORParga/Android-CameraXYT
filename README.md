# Android-CameraXYT
Aplicación sencilla que muestra como mostrar la cámara en pantalla y guardar una foto


Esta app muestra la forma más básica de capturar una imagen
desde la cámera y guardarla en un archivo

Esta app tiene dos partes fundamentales:
-Camara
-Archivo

 para el uso de la Camara, la app utiliza la Api CameraX. que es un
 conjunto de codigo que sustituye a la antigua API "Camara"
 la api oficial para manejear las camaras de un dispositivo Android

 Para poder usar la Camara en un dispositivo android, la app debe
 pedir permiso al usuario. Antiguamente, los permisos se otorgaban en
 el momento en que se instalaba la app. Por lo que el programador debía
 especificar estos permisos en el archivo AndroidManifiest.xml.
 Hoy en dia, a pesar de que el permiso se solicita en el mismo momento
 en el que la app necesita la cámara, continua siendo necesario especificarlo
 en AndroidManifest.

          <uses-feature android:name="android.hardware.camera.any"/>
          <uses-permission android:name="android.permission.CAMERA"/>

 Ademas de esta entrada en el archivo AndroidManifest, la App debe hacer una llamada
 a ActivityCompat.requestPermissions()


      ActivityCompat.requestPermissions(this,
          Constants.REQUIRED_PERMISSIONS,
          Constants.REQUEST_CODE_PERMISSIONS)

 Siendo Constants.REQUIRED_PERMISSIONS un array en el que en este caso, solo tiene un valor
 public static final String CAMERA = "android.permission.CAMERA";

 Para complicarlo un poco mas... a esta funcion solo hay que llamarla si el usuario todavía
 no ha otorgado el permiso... si tofavía no se le ha hecho la pregunta de
 si permite o no permite el uso de la camara a nuestra app
 Para saber si nuestra app tiene o no acceso a la camara se debe llamar antes de nada a la
 funcion ContextCompat.checkSelfPermission()


    private fun allPermissionGranted() =
          Constants.REQUIRED_PERMISSIONS.all{
          ContextCompat.checkSelfPermission(baseContext,it)==PackageManager.PERMISSION_GRANTED
    }

 Una vez chequeados los permisos, se puede ejecutar la funcion ActivityCompat.requestPermissions()
 la cual, interrumpe la ejecucion de la app para presentar en primer plano el famoso
 cuadro de dialogo de android preguntando al usuario si otorgao no el permiso.
 Cuando el usuario ha respondido, Android regresa a la app, ( no en el punto en el que estaba )
 sino en la funcion de tipo "Callback" definida como "onRequestPermissionsResult"... supongo que para
 complicarlo un poco mas.

       override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        if(requestCode==Constants.REQUEST_CODE_PERMISSIONS)
            if(allPermissionGranted()){

                Toast.makeText(this,"We have permission",Toast.LENGTH_SHORT).show()
            }
            else{
                Toast.makeText(this,"Permissions not granted by User",Toast.LENGTH_SHORT).show()
            }
    }

onRequestPermissionsResult() es la funcion a la que Android llama cada vez que el usuario
 pide permiso para acceder a cualquier recurso reservado... camara, internet, sistema de archivos...
así que la app tiene que comprovar a que permiso se refiere antes de presuponer nada.
 Esto lo consigue con el cíodigo definido a eleccion del programador ... que en este caso es:


      object Constants {
              ...
          const val REQUEST_CODE_PERMISSIONS = 123
              ...
      }

 Si se necesitaran más permisos, obviamente, el programador podrá elegir otros codigos a su
 gusto. Aquí se ha optado por el numero 123 pero podría haber sido cualquier numero.

 Así pues, el flujo de programa, desde que el usuario pulsa el boton "TAKE PHOTO" ,tiene dos
 ramas: directa, cuando ya tiene permiso, y otra cuando hay que preguntar. Ambas ramas desenbocan en
 la misma funcion: ImageCapture:takePicture().

 ImageCapture:takePicture() es la funcion de la API CameraX que guarda lo que muestra la camara
 en un archivo. Pero antes se debe prepar la app para que la pantalla muestre lo que la camara
 esta enfocando.
 cameraProvider.bindToLifecycle() es la funcion que asocia la camara a una superficie en la
 pantalla para mostrar lo que la camara está enfocando... y el conjunto lo asocia también
 al ciclo de vida de la app-

          cameraProvider.bindToLifecycle(
                          this,
                          cameraSelector,
                          preview,
                          imageCapture)

 camera Selector es solo una constante que indica cual de las camaras vas a usar... delantera, trasera, nocturna,
 termica, etc...

      val cameraSelector = CameraSelector.DEFAULT_BACK_CAMERA

 preview hace referencia a un control-View de la Interfaz de Usuario del tipo androidx.camera.view.PreviewView
 donde la API de muestra lo que la camara enfoca( en este caso "android:id="@+id/viewFinder")
 ... sin que el programador tenga que preocuparse de nada mas

             val preview=Preview.Builder()
                .build()
                .also { mPreview->
                    mPreview.setSurfaceProvider(binding.viewFinder.surfaceProvider)
                }

 imageCapture es el objeto al que la app hará referencia cada vez que quiera trabajar con esta camara
 que hemos "creado"... por ejemplo para tomar la foto.

          imageCapture=ImageCapture.Builder().build()

 una vez tenemos la superficie configurada para que muestre la camara en pantalla, solo queda
 tomar la foto con ImageCapture:takePicture()

           imageCapture.takePicture(
            outputOption,
            ContextCompat.getMainExecutor(this),
            object: ImageCapture.OnImageSavedCallback {
                override fun onImageSaved(outputFileResults: ImageCapture.OutputFileResults) {
                    val savedUri= Uri.fromFile(photoFile)
                    val msg ="Photo Saved!"

                    Toast.makeText(
                        this@MainActivity,
                    "$msg $savedUri",
                    Toast.LENGTH_LONG).show()
                }

                override fun onError(exception: ImageCaptureException) {
                    Log.e(Constants.TAG,
                    "onError takePicture:${exception.message}",
                    exception)
                }
            }

                )

 Esta funcion necesita tres parametros
 outputOption - hace referencia al archivo en el que se va a guardar la foto
 MainExecutor - que hace referencia al Thread principal de la aplicacion.
 OnImageSavedCallback - funcion a la que la API de CameraX llamará cuando haya guardado la foto

 para hacer el programa más sencillo, esta app usa el almacenamiento "Interno" propio de la App
 ya que este tipo de almacenamiento no necesita permisos


      private fun getOutputDirectory(): File {
          val mediaDir= externalMediaDirs.firstOrNull()?.let{
            mFile->File(mFile,resources.getString(R.string.app_name)).apply {
        mkdirs()
    }
    }
    return if(mediaDir!=null&&mediaDir.exists())
        mediaDir else filesDir
     }

 tanto filesDir como externalMediaDirs son "atajos" de kotlin para hacer mas sencilla
 la obtencion de rutas de archivo.
 en concreto filesDir... Returns the absolute path to the directory on the filesystem where files
 created with openFileOutput are stored.
 y relativo a externalMediaDirs... Returns absolute paths to application-specific directories on
 all shared/external storage devices where the application can place media files.
