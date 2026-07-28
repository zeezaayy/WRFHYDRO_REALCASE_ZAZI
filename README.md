# 🌧️ WRF-Hydro Real Case Simulation using Docker
## TC Dahlia

Dokumentasi proses pembangunan (*build*), preprocessing, dan simulasi **WRF-Hydro v5.2.0-rc1** menggunakan Docker berdasarkan studi kasus yang dikerjakan selama kegiatan magang di **Badan Riset dan Inovasi Nasional (BRIN)**.

Repository ini mencakup proses mulai dari build WRF-Hydro, pembuatan **Routing Stack** menggunakan **WRF-Hydro GIS Preprocessor**, pembuatan **LDASIN**, hingga menjalankan simulasi dan visualisasi hasil.

---

# 📑 Daftar Isi

- [Persyaratan Sistem](#-persyaratan-sistem)
- [Alur Workflow](#-alur-workflow)
- [Struktur Folder](#-struktur-folder)
- [PART I - Build WRF-Hydro](#-part-i---build-wrf-hydro)
- PART II - GIS Preprocessor
- PART III - Generate LDASIN & Run WRF-Hydro
- PART IV - Visualisasi & Troubleshooting

---

# 💻 Persyaratan Sistem

## 🛠 Software yang Digunakan

| Software | Versi |
|----------|--------|
| Docker Desktop | Latest |
| Docker Image | wrfhydro/training:v5.2.0-rc1 |
| Python | 3.11 |
| Conda | Miniconda / Anaconda |
| WRF-Hydro | v5.2.0-rc1 |
| Google Colab | Terbaru |
| QGIS | 3.x (Opsional) |

---

# 🌎 Alur Workflow

```text
ERA5
 │
 ▼
WPS
 │
 ▼
WRF
 │
 ▼
LDASIN
 │
 ▼
WRF-Hydro GIS Preprocessor
 │
 ▼
Routing Files
 │
 ▼
WRF-Hydro
 │
 ▼
CHRTOUT
LDASOUT
GWOUT
RTOUT
 │
 ▼
Visualisasi
```

Workflow di atas menunjukkan alur data mulai dari data meteorologi hingga menghasilkan output simulasi WRF-Hydro.

---

# 📂 Struktur Folder

```text
WRFHydro_Project/
│
├── trunk/
│   ├── NDHMS/
│   └── Build/
│
├── DOMAIN/
│   ├── geo_em.d03.nc
│   ├── Fulldom_hires.nc
│   ├── Route_Link.nc
│   ├── GWBUCKPARM.nc
│   ├── LakeParm.nc
│   ├── soil_properties.nc
│   ├── hydro2dtbl.nc
│   └── spatialweights.nc
│
├── FORCING/
│   └── LDASIN/
│       ├── YYYYMMDDHH00.LDASIN_DOMAIN3
│       ├── YYYYMMDDHH01.LDASIN_DOMAIN3
│       └── ...
│
├── PARAMS/
│   ├── HYDRO.TBL
│   ├── MPTABLE.TBL
│   ├── SOILPARM.TBL
│   ├── VEGPARM.TBL
│   └── GENPARM.TBL
│
├── OUTPUT/
│   ├── CHRTOUT/
│   ├── LDASOUT/
│   ├── GWOUT/
│   └── RTOUT/
│
├── LOGS/
│
├── run/
│
└── docs/
```

---

## 📁 Penjelasan Folder

| Folder | Keterangan |
|----------|------------|
| **trunk** | Source code WRF-Hydro |
| **DOMAIN** | File domain dan routing hasil preprocessing |
| **FORCING** | Data forcing meteorologi (LDASIN) |
| **PARAMS** | File parameter model |
| **OUTPUT** | Hasil simulasi WRF-Hydro |
| **LOGS** | Log proses simulasi |
| **run** | Direktori utama untuk menjalankan model |
| **docs** | Dokumentasi, gambar, GIF, dan screenshot |

---

## 📁 File pada Folder DOMAIN

| File | Fungsi | Wajib |
|------|---------|:----:|
| geo_em.d03.nc | Domain hasil WPS | ✅ |
| Fulldom_hires.nc | Domain resolusi tinggi WRF-Hydro | ✅ |
| Route_Link.nc | Informasi jaringan sungai | ✅ |
| GWBUCKPARM.nc | Parameter groundwater | ✅ |
| LakeParm.nc | Parameter danau | Opsional |
| soil_properties.nc | Parameter tanah | ✅ |
| hydro2dtbl.nc | Routing overland | ✅ |
| spatialweights.nc | Bobot spasial | ✅ |

---

# 📦 PART I — Build WRF-Hydro

Tahap ini bertujuan untuk melakukan kompilasi (*build*) source code WRF-Hydro sehingga menghasilkan file executable yang akan digunakan pada proses simulasi.

---

## 1. Pull Docker Image

Unduh image resmi WRF-Hydro.

```bash
docker pull wrfhydro/training:v5.2.0-rc1
```

Pastikan image berhasil terunduh.

```bash
docker images
```

Output yang diharapkan:

```text
REPOSITORY            TAG
wrfhydro/training     v5.2.0-rc1
```

---

## 2. Membuat Docker Container

Buat container baru.

```bash
docker run -it ^
--name wrfhydro-training ^
-v C:\WRFHydro_Project:/home/docker/WRFHydro_Project ^
wrfhydro/training:v5.2.0-rc1
```

Keterangan:

| Parameter | Fungsi |
|-----------|--------|
| `-it` | Menjalankan container secara interaktif |
| `--name` | Memberikan nama container |
| `-v` | Menghubungkan folder Windows dengan folder di dalam Docker |

---

## 3. Masuk ke Docker Container

Jika container sudah pernah dibuat sebelumnya, jalankan:

```bash
docker start wrfhydro-training

docker exec -it wrfhydro-training bash
```

---

## 4. Verifikasi Environment

Pastikan seluruh compiler tersedia.

```bash
gcc --version

gfortran --version

cmake --version

mpirun --version
```

---

## 5. Menyiapkan Source Code WRF-Hydro

Masuk ke direktori source code.

```bash
cd trunk/NDHMS
```

Buat folder build.

```bash
mkdir Build

cd Build
```

---

## 6. Build WRF-Hydro

Lakukan proses kompilasi.

```bash
cmake ..

make
```

Proses ini akan menghasilkan executable WRF-Hydro.

---

## 7. Verifikasi Hasil Build

Pastikan file berikut berhasil dibuat.

| File | Keterangan |
|------|------------|
| wrf_hydro.exe | Executable utama WRF-Hydro |
| wrf_hydro_NoahMP.exe | Executable WRF-Hydro dengan Noah-MP |

Contoh pengecekan:

```bash
ls
```

Output yang diharapkan:

```text
wrf_hydro.exe
wrf_hydro_NoahMP.exe
```

---

## ✅ Checklist PART I

| Tahapan | Status |
|----------|:------:|
| Docker Image berhasil diunduh | ✅ |
| Docker Container berhasil dibuat | ✅ |
| Berhasil masuk ke Docker | ✅ |
| Environment terverifikasi | ✅ |
| Build WRF-Hydro selesai | ✅ |
| wrf_hydro.exe berhasil dibuat | ✅ |
| wrf_hydro_NoahMP.exe berhasil dibuat | ✅ |

---

➡️ Setelah proses build selesai, langkah berikutnya adalah melakukan **GIS Preprocessor** untuk menghasilkan file routing seperti **Fulldom_hires.nc**, **Route_Link.nc**, dan file domain lainnya yang akan digunakan pada simulasi WRF-Hydro.

# 🗺️ PART II — WRF-Hydro GIS Preprocessor

Tahap ini bertujuan untuk menghasilkan **Routing Stack** yang akan digunakan sebagai input WRF-Hydro. Routing Stack dibuat menggunakan **WRF-Hydro GIS Preprocessor** berdasarkan domain WRF (`geo_em.d03.nc`) dan data DEM.

---

# 📥 Data yang Diperlukan

Sebelum memulai, siapkan file berikut.

| File | Keterangan |
|------|------------|
| geo_em.d03.nc | Domain hasil WPS |
| DEM_FULL_JAWA.tif | Digital Elevation Model (DEM) |

---

# 1. Clone WRF-Hydro GIS Preprocessor

Repository ini berisi script untuk membangun Routing Stack.

Clone repository resmi NCAR.

```bash
git clone https://github.com/NCAR/wrf_hydro_gis_preprocessor.git
```

Masuk ke folder repository.

```bash
cd wrf_hydro_gis_preprocessor
```

---

# 2. Membuat Conda Environment

Buat environment baru.

```bash
conda create -n wrfh_gis_env python=3.11
```

Aktifkan environment.

```bash
conda activate wrfh_gis_env
```

Pastikan environment berhasil dibuat.

```bash
conda env list
```

---

# 3. Install Library

Install seluruh library yang dibutuhkan.

```bash
conda install gdal
conda install rasterio
conda install geopandas
conda install whitebox
conda install xarray
conda install netcdf4
conda install numpy
conda install scipy
conda install matplotlib
```

---

## Penjelasan Library

| Library | Fungsi |
|----------|---------|
| GDAL (`osgeo`) | Membaca file DEM, GeoTIFF, dan Shapefile |
| Rasterio | Membaca data raster |
| GeoPandas | Membaca file Shapefile |
| WhiteboxTools | Membuat flow direction, flow accumulation, dan jaringan sungai |
| xarray | Membaca file NetCDF |
| netCDF4 | Membaca dan menulis file NetCDF |

---

# 4. Menyiapkan File

Salin file berikut ke folder `wrf_hydro_gis_preprocessor`.

```text
wrf_hydro_gis_preprocessor/

├── Build_Routing_Stack.py
├── geo_em.d03.nc
└── DEM_FULL_JAWA.tif
```

---

# 5. Menjalankan Build_Routing_Stack.py

Jalankan perintah berikut.

```bash
python Build_Routing_Stack.py ^
-i geo_em.d03.nc ^
-d DEM_FULL_JAWA.tif ^
-R 4 ^
-t 20 ^
-o routing_d03.zip
```

---

## Penjelasan Parameter

| Parameter | Fungsi |
|-----------|---------|
| `-i` | File domain hasil WPS |
| `-d` | File DEM |
| `-R` | Grid Refinement Ratio |
| `-t` | Stream Threshold |
| `-o` | Nama file output |

---

# 6. Hasil Routing Stack

Apabila proses berhasil, akan dihasilkan file.

```text
routing_d03.zip
```

Ekstrak file tersebut sehingga diperoleh file berikut.

```text
DOMAIN/

├── Fulldom_hires.nc
├── Route_Link.nc
├── GWBASINS.nc
├── GWBUCKPARM.nc
├── hydro2dtbl.nc
├── soil_properties.nc
├── spatialweights.nc
├── GEOGRID_LDASOUT_Spatial_Metadata.nc
└── LakeParm.nc
```

---

## Penjelasan File

| File | Fungsi |
|------|---------|
| Fulldom_hires.nc | Domain resolusi tinggi WRF-Hydro |
| Route_Link.nc | Informasi jaringan sungai |
| GWBASINS.nc | Informasi groundwater basin |
| GWBUCKPARM.nc | Parameter groundwater |
| hydro2dtbl.nc | Parameter routing permukaan |
| soil_properties.nc | Parameter tanah |
| spatialweights.nc | Bobot spasial antar grid |
| GEOGRID_LDASOUT_Spatial_Metadata.nc | Metadata spasial Noah-MP |
| LakeParm.nc | Parameter danau (opsional) |

---

# 🛠 Troubleshooting

| Error | Penyebab | Solusi | Status |
|---------|----------|---------|:------:|
| `ModuleNotFoundError: osgeo` | GDAL belum terpasang | Install GDAL (`conda install gdal`) | ✅ |
| WhiteboxTools gagal diunduh | WhiteboxTools belum tersedia | Install ulang WhiteboxTools | ✅ |
| `pkg_resources` tidak ditemukan | `setuptools` belum lengkap | Upgrade `setuptools` | ✅ |
| DEM tidak ditemukan | Lokasi DEM salah | Periksa kembali lokasi file | ✅ |
| `geo_em.d03.nc` tidak ditemukan | File domain belum disalin | Salin ulang file ke folder kerja | ✅ |
| Route_Link / Stream Shapefile belum sesuai | Jaringan sungai belum sesuai | Masih dalam proses investigasi | ⚠️ |

---

# ✅ Checklist PART II

| Tahapan | Status |
|----------|:------:|
| Clone repository GIS Preprocessor | ✅ |
| Conda Environment berhasil dibuat | ✅ |
| Library berhasil diinstal | ✅ |
| Build_Routing_Stack.py berhasil dijalankan | ✅ |
| routing_d03.zip berhasil dibuat | ✅ |
| File Routing Stack berhasil dibuat | ✅ |

---

➡️ Setelah seluruh file Routing Stack berhasil dibuat, langkah berikutnya adalah **menghasilkan file LDASIN_DOMAIN3** menggunakan Google Colab sebagai data forcing untuk menjalankan simulasi WRF-Hydro.

# 🌧️ PART III — Menyiapkan Simulasi WRF-Hydro

Setelah proses **Build WRF-Hydro** dan **Routing Stack** selesai, langkah berikutnya adalah menyiapkan data forcing (LDASIN), mengatur konfigurasi model, kemudian menjalankan simulasi WRF-Hydro.

---

# 📌 Alur Persiapan Simulasi

```text
Output WRF (wrfout)
        │
        ▼
Upload ke Google Drive
        │
        ▼
Google Colab
        │
        ▼
Membaca geo_em.d03.nc
        │
        ▼
Membaca wrfout_d03
        │
        ▼
Generate LDASIN_DOMAIN3
        │
        ▼
Copy ke Folder FORCING
        │
        ▼
Konfigurasi namelist.hrldas
        │
        ▼
Run WRF-Hydro
```

---

# 📂 File yang Harus Disiapkan

Pastikan seluruh file berikut telah tersedia sebelum menjalankan simulasi.

| Folder | File | Status |
|---------|------|:------:|
| DOMAIN | geo_em.d03.nc | ✅ |
| DOMAIN | Fulldom_hires.nc | ✅ |
| DOMAIN | Route_Link.nc | ✅ |
| DOMAIN | GWBUCKPARM.nc | ✅ |
| DOMAIN | hydro2dtbl.nc | ✅ |
| DOMAIN | soil_properties.nc | ✅ |
| DOMAIN | GEOGRID_LDASOUT_Spatial_Metadata.nc | ✅ |
| DOMAIN | wrfinput_d03 | ✅ |
| FORCING | LDASIN_DOMAIN3 | ✅ |

---

# 🌧️ Generate LDASIN

File **LDASIN_DOMAIN3** merupakan data forcing meteorologi yang digunakan oleh Noah-MP selama simulasi berlangsung.

Generate LDASIN dilakukan menggunakan **Google Colab** berdasarkan output WRF (`wrfout_d03`) dan domain WRF (`geo_em.d03.nc`).

---

## Workflow Google Colab

```text
Cell 1
Import Library
        │
        ▼
Cell 2
Mount Google Drive
        │
        ▼
Cell 3
Membaca geo_em.d03.nc
        │
        ▼
Cell 4
Membaca wrfout_d03
        │
        ▼
Cell 5
Generate LDASIN
        │
        ▼
Cell 6
Rename menjadi LDASIN_DOMAIN3
        │
        ▼
Cell 7
Simpan ke Google Drive
```

---

## Input Notebook

| File | Keterangan |
|------|------------|
| geo_em.d03.nc | Domain hasil WPS |
| wrfout_d03_* | Output simulasi WRF |
| Google Drive | Penyimpanan file |

---

## Output Notebook

```text
FORCING/

└── LDASIN/

    ├── 201711290000.LDASIN_DOMAIN3

    ├── 201711290100.LDASIN_DOMAIN3

    ├── 201711290200.LDASIN_DOMAIN3

    ├── 201711290300.LDASIN_DOMAIN3

    └── ...
```

---

## Variabel pada LDASIN

| Variabel | Keterangan |
|----------|------------|
| T2D | Temperatur udara 2 meter |
| Q2D | Specific Humidity |
| PSFC | Tekanan permukaan |
| U2D | Angin arah zonal |
| V2D | Angin arah meridional |
| SWDOWN | Radiasi gelombang pendek |
| LWDOWN | Radiasi gelombang panjang |
| RAINRATE | Curah hujan |

---

## Menyalin File LDASIN

Setelah seluruh file LDASIN berhasil dibuat, salin seluruh file ke folder berikut.

```text
run/

├── FORCING/

│   └── LDASIN/

│       ├── 201711290000.LDASIN_DOMAIN3

│       ├── 201711290100.LDASIN_DOMAIN3

│       └── ...
```

> 💡 **Catatan**
>
> Pastikan jumlah file LDASIN sesuai dengan durasi simulasi (`KHOUR = 95`). Apabila jumlah file forcing tidak lengkap, simulasi dapat berhenti sebelum waktu akhir.

---

# ⚙️ Konfigurasi namelist.hrldas

Sebelum menjalankan model, lakukan konfigurasi pada file **namelist.hrldas**.

## Bagian HYDRO

```fortran
&HYDRO_nlist

sys_cpl = 1

GEO_STATIC_FLNM = "./DOMAIN/geo_em.d03.nc"
GEO_FINEGRID_FLNM = "./DOMAIN/Fulldom_hires.nc"
HYDROTBL_F = "./DOMAIN/hydro2dtbl.nc"
LAND_SPATIAL_META_FLNM = "./DOMAIN/GEOGRID_LDASOUT_Spatial_Metadata.nc"

IGRID = 3

rst_dt = -99999
rst_typ = 0

rst_bi_in = 0
rst_bi_out = 0

RSTRT_SWC = 0
GW_RESTART = 0

out_dt = 60

SPLIT_OUTPUT_COUNT = 1

order_to_write = 1

io_form_outputs = 1
io_config_outputs = 1

t0OutputFlag = 1

output_channelBucket_influx = 0

CHRTOUT_DOMAIN = 1
CHANOBS_DOMAIN = 0
CHRTOUT_GRID = 0
LSMOUT_DOMAIN = 0
RTOUT_DOMAIN = 1
output_gw = 1
outlake = 0
frxst_pts_out = 0

NSOIL = 4

ZSOIL8(1) = -0.10
ZSOIL8(2) = -0.40
ZSOIL8(3) = -1.00
ZSOIL8(4) = -2.00

DXRT = 833.333
AGGFACTRT = 4

DTRT_CH = 150
DTRT_TER = 720

SUBRTSWCRT = 1
OVRTSWCRT = 1

rt_option = 1

CHANRTSWCRT = 1
channel_option = 2

route_link_f = "./DOMAIN/Route_Link.nc"

compound_channel = .FALSE.

GWBASESWCRT = 1

gwbasmskfil = "./DOMAIN/GWBASINS.nc"

GWBUCKPARM_file = "./DOMAIN/GWBUCKPARM.nc"

UDMP_OPT = 0

/
```

---

## Bagian NUDGING

```fortran
&NUDGING_nlist

timeSlicePath = "./nudgingTimeSliceObs/"

nudgingParamFile = "DOMAIN/nudgingParams.nc"

readTimesliceParallel = .TRUE.

temporalPersistence = .FALSE.

nLastObs = 960

persistBias = .FALSE.

biasWindowBeforeT0 = .FALSE.

maxAgePairsBiasPersist = -960

minNumPairsBiasPersist = 8

invDistTimeWeightBias = .TRUE.

noConstInterfBias = .FALSE.

/
```

---

## Bagian NOAHLSM_OFFLINE

Konfigurasi berikut digunakan untuk mengatur model Noah-MP, lokasi forcing, waktu simulasi, serta interval output.

```fortran
&NOAHLSM_OFFLINE

HRLDAS_SETUP_FILE = "./DOMAIN/wrfinput_d03"
INDIR             = "./FORCING"
SPATIAL_FILENAME  = "./DOMAIN/soil_properties.nc"
OUTDIR            = "./"

START_YEAR        = 2017
START_MONTH       = 11
START_DAY         = 29
START_HOUR        = 00
START_MIN         = 00

KHOUR             = 95

! Physics options
DYNAMIC_VEG_OPTION                = 4
CANOPY_STOMATAL_RESISTANCE_OPTION = 1
BTR_OPTION                        = 1
RUNOFF_OPTION                     = 3
SURFACE_DRAG_OPTION               = 1
FROZEN_SOIL_OPTION                = 1
SUPERCOOLED_WATER_OPTION          = 1
RADIATIVE_TRANSFER_OPTION         = 3
SNOW_ALBEDO_OPTION                = 2
PCP_PARTITION_OPTION              = 1
TBOT_OPTION                       = 2
TEMP_TIME_SCHEME_OPTION           = 3
GLACIER_OPTION                    = 2
SURFACE_RESISTANCE_OPTION         = 4

FORCING_TIMESTEP = 3600
NOAH_TIMESTEP    = 3600
OUTPUT_TIMESTEP  = 3600

RESTART_FREQUENCY_HOURS = -99999

SPLIT_OUTPUT_COUNT = 1

NSOIL = 4

soil_thick_input(1) = 0.10
soil_thick_input(2) = 0.30
soil_thick_input(3) = 0.60
soil_thick_input(4) = 1.00

ZLVL = 10.0

rst_bi_in = 0
rst_bi_out = 0

/

&WRF_HYDRO_OFFLINE

FORC_TYP = 2

/
```

---

## 📖 Penjelasan Parameter Penting

| Parameter | Nilai | Keterangan |
|-----------|-------|------------|
| `HRLDAS_SETUP_FILE` | `wrfinput_d03` | File inisialisasi hasil WRF |
| `INDIR` | `./FORCING` | Lokasi file LDASIN |
| `SPATIAL_FILENAME` | `soil_properties.nc` | Parameter tanah |
| `OUTDIR` | `./` | Lokasi output simulasi |
| `START_YEAR` | 2017 | Tahun awal simulasi |
| `START_MONTH` | 11 | Bulan awal simulasi |
| `START_DAY` | 29 | Tanggal awal simulasi |
| `START_HOUR` | 00 | Jam awal simulasi |
| `KHOUR` | 95 | Lama simulasi (95 jam) |
| `FORCING_TIMESTEP` | 3600 | Interval forcing (1 jam) |
| `NOAH_TIMESTEP` | 3600 | Time step Noah-MP |
| `OUTPUT_TIMESTEP` | 3600 | Interval output (1 jam) |
| `FORC_TYP` | 2 | Menggunakan forcing bertipe LDASIN |

---

# ✅ Checklist Sebelum Simulasi

Sebelum menjalankan WRF-Hydro, pastikan seluruh file telah tersedia.

| Folder | File | Status |
|---------|------|:------:|
| DOMAIN | geo_em.d03.nc | ✅ |
| DOMAIN | Fulldom_hires.nc | ✅ |
| DOMAIN | Route_Link.nc | ✅ |
| DOMAIN | GWBUCKPARM.nc | ✅ |
| DOMAIN | hydro2dtbl.nc | ✅ |
| DOMAIN | soil_properties.nc | ✅ |
| DOMAIN | GEOGRID_LDASOUT_Spatial_Metadata.nc | ✅ |
| DOMAIN | wrfinput_d03 | ✅ |
| FORCING | Seluruh file LDASIN_DOMAIN3 | ✅ |
| PARAMS | HYDRO.TBL | ✅ |
| PARAMS | SOILPARM.TBL | ✅ |
| PARAMS | VEGPARM.TBL | ✅ |
| PARAMS | MPTABLE.TBL | ✅ |

---

# 🚀 Menjalankan Simulasi

Masuk ke direktori **run**.

```bash
cd run
```

Jalankan model.

```bash
./wrf_hydro_NoahMP.exe
```

Apabila build menggunakan MPI.

```bash
mpirun -np 4 ./wrf_hydro.exe
```

---

## 📋 Verifikasi Simulasi

Selama simulasi berlangsung, periksa log untuk memastikan model berjalan tanpa error.

Contoh:

```bash
tail -f LOGS/run_realcase.log
```

Apabila simulasi selesai dengan baik, proses akan berhenti tanpa muncul pesan **FATAL ERROR** maupun **MPI_ABORT**.

---

# 📤 Output Simulasi

Setelah simulasi selesai, beberapa folder output akan terbentuk secara otomatis.

```text
OUTPUT/

├── CHRTOUT/
├── LDASOUT/
├── GWOUT/
└── RTOUT/
```

---

## Penjelasan Output

| Folder | Keterangan |
|----------|------------|
| CHRTOUT | Output simulasi aliran sungai (Channel Routing) |
| LDASOUT | Output Noah-MP (Land Surface Model) |
| GWOUT | Output groundwater |
| RTOUT | Output river routing |

---

## Variabel yang Umum Digunakan

### CHRTOUT

| Variabel | Keterangan |
|-----------|------------|
| streamflow | Debit sungai |
| velocity | Kecepatan aliran |
| water_depth | Kedalaman air |

---

### LDASOUT

| Variabel | Keterangan |
|-----------|------------|
| T2D | Temperatur udara 2 meter |
| TSK | Temperatur permukaan |
| SOIL_M | Kelembapan tanah |
| SOIL_T | Temperatur tanah |
| SFCRNOFF | Limpasan permukaan |
| UDRNOFF | Limpasan bawah permukaan |

---

### GWOUT

| Variabel | Keterangan |
|-----------|------------|
| groundwater_storage | Penyimpanan airtanah |
| water_table_depth | Kedalaman muka airtanah |
| baseflow | Aliran dasar |

---

### RTOUT

| Variabel | Keterangan |
|-----------|------------|
| discharge | Debit sungai |
| storage | Penyimpanan aliran |
| inflow | Debit masuk |
| outflow | Debit keluar |

---

## Verifikasi Output

Pastikan seluruh folder output berhasil dibuat.

```bash
tree .
```

atau

```bash
ls CHRTOUT
ls LDASOUT
ls GWOUT
ls RTOUT
```

---

## 💡 Tips

Sebelum melakukan visualisasi, buka salah satu file NetCDF menggunakan `ncdump` atau Python untuk memastikan variabel telah tersimpan dengan benar.

```bash
ncdump -h CHRTOUT/CHRTOUT_DOMAIN1
```

---

## 🛠 Troubleshooting

| Error | Penyebab | Fungsi/Fase | Solusi | Status |
|---------|----------|------------|----------|:------:|
| LDASIN tidak ditemukan | File belum berada di folder `FORCING` | Run WRF-Hydro | Salin ulang seluruh file LDASIN | ✅ |
| Jumlah LDASIN tidak sesuai | Periode forcing belum lengkap | Generate LDASIN | Generate ulang menggunakan notebook Colab | ✅ |
| Konfigurasi `namelist.hrldas` salah | Lokasi file atau waktu simulasi tidak sesuai | Run WRF-Hydro | Periksa kembali `INDIR`, `OUTDIR`, `START_*`, dan `KHOUR` | ✅ |
| `MPI_ABORT` | Konfigurasi domain atau routing belum sesuai | Run WRF-Hydro | Periksa kembali file routing dan konfigurasi model | ✅ |

---

## ✅ Checklist PART III

| Tahapan | Status |
|----------|:------:|
| Routing Stack selesai | ✅ |
| LDASIN berhasil dibuat | ✅ |
| LDASIN berhasil disalin | ✅ |
| `namelist.hrldas` selesai dikonfigurasi | ✅ |
| Simulasi WRF-Hydro berhasil dijalankan | ✅ |
| Folder output berhasil dibuat | ✅ |

---

➡️ Tahap berikutnya adalah melakukan **visualisasi hasil simulasi**, pengecekan kualitas output, serta dokumentasi hasil menggunakan Python, Google Colab, atau QGIS.

# 📊 PART IV — Visualisasi, Hasil Simulasi, dan Troubleshooting

Tahap terakhir adalah melakukan visualisasi hasil simulasi WRF-Hydro serta memastikan seluruh output telah berhasil dihasilkan dengan benar.

Visualisasi dapat dilakukan menggunakan **Python**, **Google Colab**, maupun **QGIS** sesuai kebutuhan analisis.

---

# 📈 Alur Visualisasi

```text
CHRTOUT
LDASOUT
GWOUT
RTOUT
      │
      ▼
Python / Google Colab
      │
      ▼
Membaca File NetCDF
      │
      ▼
Visualisasi
      │
      ▼
GIF / PNG / Grafik
```

---

# 📂 Struktur Output

```text
OUTPUT/

├── CHRTOUT/
│
├── LDASOUT/
│
├── GWOUT/
│
└── RTOUT/
```

---

# 🌊 CHRTOUT

Folder **CHRTOUT** berisi hasil simulasi aliran sungai (*Channel Routing*).

### Variabel yang Umum Digunakan

| Variabel | Keterangan |
|-----------|------------|
| streamflow | Debit sungai |
| velocity | Kecepatan aliran |
| water_depth | Kedalaman air |

---

# 🌱 LDASOUT

Folder **LDASOUT** berisi output model Noah-MP.

### Variabel yang Umum Digunakan

| Variabel | Keterangan |
|-----------|------------|
| T2D | Temperatur udara 2 meter |
| TSK | Temperatur permukaan |
| SOIL_M | Kelembapan tanah |
| SOIL_T | Temperatur tanah |
| SFCRNOFF | Limpasan permukaan |
| UDRNOFF | Limpasan bawah permukaan |

---

# 💧 GWOUT

Folder **GWOUT** berisi hasil simulasi groundwater.

### Variabel yang Umum Digunakan

| Variabel | Keterangan |
|-----------|------------|
| groundwater_storage | Penyimpanan airtanah |
| water_table_depth | Kedalaman muka airtanah |
| baseflow | Aliran dasar |

---

# 🌊 RTOUT

Folder **RTOUT** berisi hasil river routing.

### Variabel yang Umum Digunakan

| Variabel | Keterangan |
|-----------|------------|
| discharge | Debit sungai |
| storage | Penyimpanan aliran |
| inflow | Debit masuk |
| outflow | Debit keluar |

---

# 🎥 Hasil Simulasi

## Animasi WRF-Hydro

Tambahkan GIF hasil simulasi di bawah ini.

<p align="center">
<img src="docs/wrfhydro.gif" width="900">
</p>

---

## Visualisasi Output

Tambahkan screenshot hasil visualisasi.

<p align="center">
<img src="docs/output.png" width="900">
</p>

---

## Contoh Analisis

Visualisasi dapat dilakukan untuk beberapa variabel berikut.

| Output | Contoh Visualisasi |
|----------|-------------------|
| CHRTOUT | Debit sungai |
| LDASOUT | Temperatur permukaan |
| LDASOUT | Soil Moisture |
| LDASOUT | Surface Runoff |
| GWOUT | Groundwater |
| RTOUT | River Routing |

---

# 📁 Folder Dokumentasi

Disarankan membuat folder berikut.

```text
docs/

├── workflow.png

├── routing_stack.png

├── wrfhydro.gif

├── output.png

└── comparison.png
```

---

# 🛠 Troubleshooting

Selama proses pengerjaan, beberapa kendala yang ditemui beserta solusinya dirangkum pada tabel berikut.

| Error | Penyebab | Fungsi/Fase | Solusi | Status |
|---------|----------|------------|----------|:------:|
| ModuleNotFoundError: osgeo | Library GDAL belum terpasang | GIS Preprocessor | Install GDAL (`conda install gdal`) | ✅ |
| WhiteboxTools gagal diunduh | WhiteboxTools belum tersedia | GIS Preprocessor | Install ulang WhiteboxTools | ✅ |
| pkg_resources tidak ditemukan | `setuptools` belum lengkap | Build_Routing_Stack.py | Upgrade `setuptools` | ✅ |
| LDASIN belum sesuai | File forcing belum lengkap | Generate LDASIN | Generate ulang menggunakan notebook Colab | ✅ |
| MPI_ABORT | Konfigurasi domain atau routing belum sesuai | Run WRF-Hydro | Periksa kembali `namelist.hrldas` dan file routing | ✅ |
| Route_Link / Stream Shapefile | Jaringan sungai belum sesuai | Routing Stack | Masih dalam proses investigasi | ⚠️ |

---

# 📋 Progress Pengerjaan

| Tahapan | Status |
|----------|:------:|
| Build WRF-Hydro | ✅ |
| Build Noah-MP | ✅ |
| GIS Preprocessor | ✅ |
| Routing Stack | ✅ |
| Generate LDASIN | ✅ |
| Konfigurasi namelist | ✅ |
| Run WRF-Hydro | ✅ |
| Visualisasi | ✅ |
| Route_Link / Stream Shapefile | ⚠️ |

---

# 🚧 Pengembangan Selanjutnya

Beberapa pengembangan yang direncanakan pada repository ini antara lain:

- Perbaikan Route_Link dan Stream Shapefile.
- Penambahan notebook validasi hasil simulasi.
- Penambahan notebook visualisasi CHRTOUT.
- Penambahan notebook visualisasi LDASOUT.
- Perbandingan hasil WRF-Hydro dengan data observasi.
- Perhitungan RMSE dan SSIM.
- Dokumentasi proses visualisasi menggunakan Google Colab.

---

# 📚 Referensi

| Referensi | Keterangan |
|-----------|------------|
| WRF-Hydro Documentation | Dokumentasi resmi |
| WRF-Hydro GIS Preprocessor | Routing Stack |
| Docker | Container |
| Google Colab | Generate LDASIN |

---

