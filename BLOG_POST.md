## BMI Calculator

---

Pada pertemuan kali ini kita ditugaskan membuat program BMI Calculator sederhana menggunakan Jetpack Compose, mengikuti referensi desain UI yang sudah diberikan.

https://github.com/remonilo/ppb-m-tugas6-bmicalculator

Untuk program ini, inti logikanya ada di fungsi `hitungBmi()` yang mengambil input berat dan tinggi badan, lalu menghitung IMT dengan rumus `berat / tinggi(m)²`. Hasilnya dipetakan ke kategori (Kurus, Normal, Gemuk, Obesitas) lewat fungsi `categoryFor()`, masing-masing dengan warna badge berbeda.

```kotlin
fun hitungBmi() {
    val berat = beratBadan.toDoubleOrNull()
    val tinggiCm = tinggiBadan.toDoubleOrNull()
    if (berat == null || tinggiCm == null || berat <= 0 || tinggiCm <= 0) {
        errorMessage = "Masukkan berat dan tinggi badan yang valid"
        bmiResult = null
        return
    }
    val tinggiM = tinggiCm / 100.0
    bmiResult = berat / (tinggiM * tinggiM)
    errorMessage = null
}
```

Selain tombol Hitung, ada tombol Reset yang mengosongkan kembali seluruh state, dan tombol Lihat Kategori yang membuka `AlertDialog` berisi daftar rentang kategori BMI. Kartu hasil hanya muncul setelah `bmiResult` terisi, memakai `bmiResult?.let { }` di Compose.

Setelah dijalankan, program menampilkan input berat/tinggi, tiga tombol aksi, dan kartu hasil BMI beserta kategorinya seperti pada gambar terlampir.

!image.png

### Sumber Kode

---

```kotlin
package com.remonilo.bmicalculator

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.foundation.verticalScroll
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Button
import androidx.compose.material3.ButtonDefaults
import androidx.compose.material3.Card
import androidx.compose.material3.CardDefaults
import androidx.compose.material3.OutlinedButton
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.OutlinedTextFieldDefaults
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.KeyboardType
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.remonilo.bmicalculator.ui.theme.BMICalculatorTheme
import java.util.Locale

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BMICalculatorTheme {
                BmiCalculatorApp()
            }
        }
    }
}

private val HeaderBlue = Color(0xFF2F6FE0)
private val PurplePrimary = Color(0xFF6C4CE0)
private val GreenPrimary = Color(0xFF3BB77E)
private val FieldBackground = Color(0xFFEFF2F9)
private val ScreenBackground = Color(0xFFF6F8FC)

data class BmiCategory(
    val label: String,
    val range: String,
    val color: Color
)

private val bmiCategories = listOf(
    BmiCategory("Kurus", "< 18.5", Color(0xFF4C8DFF)),
    BmiCategory("Normal", "18.5 - 24.9", GreenPrimary),
    BmiCategory("Gemuk", "25.0 - 29.9", Color(0xFFF2A93B)),
    BmiCategory("Obesitas", "\u2265 30.0", Color(0xFFE35D5D))
)

private fun categoryFor(bmi: Double): BmiCategory = when {
    bmi < 18.5 -> bmiCategories[0]
    bmi < 25.0 -> bmiCategories[1]
    bmi < 30.0 -> bmiCategories[2]
    else -> bmiCategories[3]
}

@Preview(showBackground = true)
@Composable
fun BmiCalculatorApp() {
    var beratBadan by rememberSaveable { mutableStateOf("70") }
    var tinggiBadan by rememberSaveable { mutableStateOf("170") }
    var bmiResult by rememberSaveable { mutableStateOf<Double?>(null) }
    var errorMessage by rememberSaveable { mutableStateOf<String?>(null) }
    var showKategoriDialog by remember { mutableStateOf(false) }

    fun hitungBmi() {
        val berat = beratBadan.toDoubleOrNull()
        val tinggiCm = tinggiBadan.toDoubleOrNull()
        if (berat == null || tinggiCm == null || berat <= 0 || tinggiCm <= 0) {
            errorMessage = "Masukkan berat dan tinggi badan yang valid"
            bmiResult = null
            return
        }
        val tinggiM = tinggiCm / 100.0
        bmiResult = berat / (tinggiM * tinggiM)
        errorMessage = null
    }

    fun reset() {
        beratBadan = ""
        tinggiBadan = ""
        bmiResult = null
        errorMessage = null
    }

    Surface(modifier = Modifier.fillMaxSize(), color = ScreenBackground) {
        Column(modifier = Modifier.fillMaxSize()) {
            BmiHeader()
            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .verticalScroll(rememberScrollState())
                    .padding(horizontal = 20.dp, vertical = 20.dp),
                verticalArrangement = Arrangement.spacedBy(14.dp)
            ) {
                BmiInputField(
                    label = "Berat Badan (kg)",
                    value = beratBadan,
                    onValueChange = { beratBadan = it }
                )
                BmiInputField(
                    label = "Tinggi Badan (cm)",
                    value = tinggiBadan,
                    onValueChange = { tinggiBadan = it }
                )

                errorMessage?.let { msg ->
                    Text(text = msg, color = Color(0xFFE35D5D), fontSize = 13.sp)
                }

                Button(
                    onClick = { hitungBmi() },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(52.dp),
                    shape = RoundedCornerShape(14.dp),
                    colors = ButtonDefaults.buttonColors(containerColor = PurplePrimary)
                ) {
                    Text("Hitung BMI", fontWeight = FontWeight.SemiBold)
                }

                OutlinedButton(
                    onClick = { reset() },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(52.dp),
                    shape = RoundedCornerShape(14.dp),
                    border = androidx.compose.foundation.BorderStroke(1.5.dp, HeaderBlue)
                ) {
                    Text("Reset", color = HeaderBlue, fontWeight = FontWeight.SemiBold)
                }

                Button(
                    onClick = { showKategoriDialog = true },
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(52.dp),
                    shape = RoundedCornerShape(14.dp),
                    colors = ButtonDefaults.buttonColors(containerColor = GreenPrimary)
                ) {
                    Text("Lihat Kategori", fontWeight = FontWeight.SemiBold)
                }

                bmiResult?.let { hasil ->
                    Spacer(modifier = Modifier.height(4.dp))
                    HasilBmiCard(bmi = hasil)
                }
            }
        }
    }

    if (showKategoriDialog) {
        KategoriBmiDialog(onDismiss = { showKategoriDialog = false })
    }
}

@Composable
private fun BmiHeader() {
    Surface(color = HeaderBlue, modifier = Modifier.fillMaxWidth()) {
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(horizontal = 20.dp, vertical = 16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            Box(
                modifier = Modifier
                    .size(32.dp)
                    .background(Color.White, CircleShape),
                contentAlignment = Alignment.Center
            ) {
                Text("\u2696\uFE0F", fontSize = 16.sp)
            }
            Spacer(modifier = Modifier.width(12.dp))
            Text(
                text = "BMI Calculator",
                color = Color.White,
                fontSize = 18.sp,
                fontWeight = FontWeight.Bold,
                modifier = Modifier.weight(1f)
            )
            Text(text = "+", color = Color.White, fontSize = 22.sp, fontWeight = FontWeight.Bold)
        }
    }
}

@Composable
private fun BmiInputField(
    label: String,
    value: String,
    onValueChange: (String) -> Unit
) {
    Column {
        Text(
            text = label,
            fontSize = 13.sp,
            fontWeight = FontWeight.Medium,
            color = Color(0xFF6B7280)
        )
        Spacer(modifier = Modifier.height(6.dp))
        OutlinedTextField(
            value = value,
            onValueChange = onValueChange,
            modifier = Modifier.fillMaxWidth(),
            shape = RoundedCornerShape(12.dp),
            singleLine = true,
            keyboardOptions = KeyboardOptions(keyboardType = KeyboardType.Decimal),
            colors = OutlinedTextFieldDefaults.colors(
                focusedContainerColor = FieldBackground,
                unfocusedContainerColor = FieldBackground,
                focusedBorderColor = Color.Transparent,
                unfocusedBorderColor = Color.Transparent
            )
        )
    }
}

@Composable
private fun HasilBmiCard(bmi: Double) {
    val kategori = categoryFor(bmi)
    Card(
        modifier = Modifier.fillMaxWidth(),
        shape = RoundedCornerShape(16.dp),
        colors = CardDefaults.cardColors(containerColor = Color.White),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp)
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(text = "Hasil BMI", fontSize = 14.sp, color = Color(0xFF6B7280))
            Spacer(modifier = Modifier.height(6.dp))
            Text(
                text = String.format(Locale.US, "%.1f", bmi),
                fontSize = 34.sp,
                fontWeight = FontWeight.Bold
            )
            Spacer(modifier = Modifier.height(10.dp))
            Box(
                modifier = Modifier
                    .background(kategori.color, RoundedCornerShape(50))
                    .padding(horizontal = 18.dp, vertical = 6.dp)
            ) {
                Text(text = kategori.label, color = Color.White, fontWeight = FontWeight.SemiBold)
            }
        }
    }
}

@Composable
private fun KategoriBmiDialog(onDismiss: () -> Unit) {
    AlertDialog(
        onDismissRequest = onDismiss,
        confirmButton = {
            Button(onClick = onDismiss, colors = ButtonDefaults.buttonColors(containerColor = PurplePrimary)) {
                Text("Tutup")
            }
        },
        title = { Text("Kategori BMI", fontWeight = FontWeight.Bold) },
        text = {
            Column(verticalArrangement = Arrangement.spacedBy(10.dp)) {
                bmiCategories.forEach { kategori ->
                    Row(verticalAlignment = Alignment.CenterVertically) {
                        Box(
                            modifier = Modifier
                                .size(12.dp)
                                .background(kategori.color, CircleShape)
                        )
                        Spacer(modifier = Modifier.width(10.dp))
                        Text(text = kategori.label, fontWeight = FontWeight.Medium, modifier = Modifier.weight(1f))
                        Text(text = kategori.range, color = Color(0xFF6B7280), fontSize = 13.sp)
                    }
                }
            }
        }
    )
}
```
