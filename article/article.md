## > Introdução

WorkManager é a biblioteca do Android que você usa quando precisa executar tarefas em segundo plano de forma confiável. Ele lida com toda a complexidade de manter suas tarefas rodando, mesmo que o aplicativo seja fechado ou o dispositivo reiniciado. Basicamente, você configura uma tarefa, define quando ela deve rodar e o WorkManager cuida do resto. Ele é perfeito para qualquer coisa que não precisa ser feita imediatamente e pode esperar um pouco para ser executada.

Para seguir os exemplos dos tópicos abaixo, realize as seguintes configurações no seu projeto de exemplo:

### 1. Adicionar Dependências

Primeiro, adicione a dependência do WorkManager ao seu arquivo `build.gradle`:

```groovy
dependencies {
    implementation "androidx.work:work-runtime-ktx:2.7.1"
}
```

Segundo, adicione as permissões necessárias ao seu arquivo `AndroidManifest.xml` para garantir que o WorkManager possa funcionar corretamente:

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

## > Sincronização de Dados em Segundo Plano

Imagine que você tem um app de notas e quer que tudo o que o usuário escreve seja sincronizado com a nuvem, mas sem atrapalhar a experiência dele. É aí que o WorkManager brilha! Você configura uma tarefa de sincronização que roda em segundo plano, garantindo que, mesmo se o app for fechado, as notas sejam sincronizadas quando o dispositivo estiver ocioso e conectado à internet. Assim, os dados do usuário estão sempre atualizados sem ele perceber.


### 1. Criar a Classe do Worker

Vamos criar uma classe que estende `Worker` e define o que será feito durante a sincronização das notas.

```kotlin
import android.content.Context
import androidx.work.Worker
import androidx.work.WorkerParameters

class SyncNotesWorker(appContext: Context, workerParams: WorkerParameters):
    Worker(appContext, workerParams) {

    override fun doWork(): Result {
        return try {
            // Aqui você coloca o código para sincronizar as notas com a nuvem
            syncNotesWithCloud()
            Result.success()
        } catch (e: Exception) {
            Result.retry()
        }
    }

    private fun syncNotesWithCloud() {
        // Simulação da sincronização com a nuvem
        // Aqui você colocaria o código real para enviar as notas ao servidor
    }
}
```

### 2. Agendar o WorkRequest

Agora, agende o WorkRequest para sincronizar as notas. Você pode fazer isso no local apropriado do seu código, como no `onCreate` do seu `MainActivity` ou em um lugar onde a sincronização precisa ser iniciada.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.work.Constraints
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkManager
import java.util.concurrent.TimeUnit

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Configurar as restrições (somente sincronizar quando o dispositivo estiver ocioso e conectado à internet)
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .setRequiresBatteryNotLow(true)
            .setRequiresCharging(false)
            .setRequiresDeviceIdle(true)
            .build()

        // Criar o WorkRequest
        val syncWorkRequest = OneTimeWorkRequestBuilder<SyncNotesWorker>()
            .setConstraints(constraints)
            .setInitialDelay(10, TimeUnit.MINUTES) // Exemplo de delay inicial
            .build()

        // Enviar a tarefa ao WorkManager
        WorkManager.getInstance(this).enqueue(syncWorkRequest)
    }
}
```

Pronto! Com esse código, você configurou o WorkManager para sincronizar as notas com a nuvem em segundo plano, respeitando as condições de ociosidade do dispositivo e conexão à internet.

## > Processamento de Imagens e Arquivos em Segundo Plano

Vamos supor que seu app permita que os usuários façam upload de fotos ou vídeos. Essas tarefas podem ser demoradas e consumir bastante recursos, então você não quer que isso aconteça enquanto o usuário está tentando usar o app. Com o WorkManager, você pode configurar essas tarefas para rodarem em segundo plano, garantindo que os uploads aconteçam de forma suave, sem interromper a usabilidade do app. Ele até lida com o reenvio automático em caso de falhas na rede!

### 1. Criar a Classe do Worker

Vamos criar uma classe que estende `Worker` e define o que será feito durante o upload das fotos ou vídeos.

```kotlin
import android.content.Context
import androidx.work.Worker
import androidx.work.WorkerParameters
import java.io.File

class UploadMediaWorker(appContext: Context, workerParams: WorkerParameters):
    Worker(appContext, workerParams) {

    override fun doWork(): Result {
        val fileUri = inputData.getString("file_uri")
        return try {
            if (fileUri != null) {
                uploadFile(File(fileUri))
                Result.success()
            } else {
                Result.failure()
            }
        } catch (e: Exception) {
            Result.retry()
        }
    }

    private fun uploadFile(file: File) {
        // Simulação do upload do arquivo
        // Aqui você colocaria o código real para enviar o arquivo ao servidor
    }
}
```

### 2. Agendar o WorkRequest

Agora, agende o WorkRequest para fazer o upload dos arquivos. Você pode fazer isso no local apropriado do seu código, como no `onCreate` do seu `MainActivity` ou em um lugar onde o upload precisa ser iniciado.

```kotlin
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.work.Constraints
import androidx.work.Data
import androidx.work.OneTimeWorkRequestBuilder
import androidx.work.WorkManager
import androidx.work.NetworkType
import java.util.concurrent.TimeUnit

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val fileUri = "path/to/your/file" // Substitua pelo caminho real do arquivo

        // Configurar as restrições (somente fazer upload quando o dispositivo estiver conectado à internet)
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.CONNECTED)
            .build()

        // Criar o WorkRequest e passar o URI do arquivo como dado de entrada
        val uploadWorkRequest = OneTimeWorkRequestBuilder<UploadMediaWorker>()
            .setConstraints(constraints)
            .setInputData(
                Data.Builder()
                    .putString("file_uri", fileUri)
                    .build()
            )
            .setInitialDelay(1, TimeUnit.MINUTES) // Exemplo de delay inicial
            .build()

        // Enviar a tarefa ao WorkManager
        WorkManager.getInstance(this).enqueue(uploadWorkRequest)
    }
}
```

Pronto! Com esse código, você configurou o WorkManager para fazer upload de fotos ou vídeos em segundo plano, respeitando as condições de conexão à internet. Se o upload falhar devido a problemas na rede, o WorkManager tentará novamente automaticamente.

## > Conclusão

O artigo acima foi criado com ferramentas de LLM e revisado por um humano, dúvidas e sugestões são bem vindas!