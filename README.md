# LABORATORIO 5

![](https://github.com/gaby2804/LABFINAL/blob/main/WhatsApp%20Image%202025-05-15%20at%2011.40.13%20AM.jpeg)

## Introduccion
La variabilidad de la frecuencia cardiaca (HRV) es un parámetro esencial para verificar la actividad del sistema nervioso autónomo y como afecta al ritmo cardiaco. Al analizar la HRV se evalúa la homeostasis que mantienen la actividad simpática y parasimpática, mostrando como se encuentra fisiológicamente el sistema y su capacidad de adaptación a disferentes estímulos, de esta manera identificar las fluctuaciones que hay en las frecuencias cardiacas de la HRV.

## Procedimiento
Este laboratorio fue diseñado para analizar como varia la frecuencia cardiaca (HRV) por medio de la transformada de Wavelet, con el fin de identificar cambios en las frecuencias de la señal capturada y estudiar la variabilidad que tiene en el dominio del tiempo. Permitiendo evaluar como se encuentra el funcionamiento del sistema nervioso autónomo que se compone del sistema nervioso simpático y parasimpático, esto gracias a las fluctuaciones que se evidencien en los intervalos RR que se capten en el ECG.

## Objetivo
Analizar la variabilidad de la frecuencia cardíaca (HRV) utilizando la
transformada wavelet para identificar cambios en las frecuencias características
y analizar la dinámica temporal de la señal cardíaca.

## Fundamento teorico
El sistema nervioso autónomo equilibra las funciones involuntarias del cuerpo y está conformado por dos partes principales: el sistema simpático y el sistema parasimpático, el primero prepara al cuerpo en situaciones de estrés y aumenta la frecuencia cardiaca, la actividad parasimpática hace que el cuerpo se relaje y tiene un efecto contrario en la frecuencia cardiaca (Guyton & Hall, 2011).

La frecuencia cardiaca como fue anteriormente mencionado es afectada por el sistema nervioso autónomo, el sistema simpático acelera el ritmo cardiaco debido a la liberación de noradrenalina, y el parasimpático lo vuelve más lento por la acción del nervio vago, además de la liberación de acetilcolina (Berntson et al., 1997). Esta modulación es importante para que el organismo se pueda adaptar a las necesidades a las que sea expuesto.

La HRV son las fluctuaciones que tiene la señal en un intervalo de tiempo que hay entre latidos cardiacos secuenciales o llamados R-R. Estas diferencias muestran la capacidad del sistema nervioso a la hora de responder cambios tanto internos como externos. Cuando hay una mayor HRV es posible que se deba a un sistema cardiovascular sano y con control automático (Task Force, 1996).

La transformada wavelet es una herramienta que nos permite conocer las señales que no son estacionarias, como el ECG, en el que se proporciona información en el dominio del tiempo y de la frecuencia (Mallat, 1999). A partir de esta técnica se pueden identificar como varían las frecuencias características en el tiempo además de su acción en bandas de frecuencia especificas (Adisson, 2005) 
## Adquisicion de la señal
Se realizó una adquisición de datos utilizando una tarjeta NI DAQ mediante la biblioteca nidaqmx. Se configuró una entrada analógica (Dev1/ai0) con una frecuencia de muestreo de 250 Hz durante un periodo de 5 minutos. Los datos fueron leídos en modo continuo y almacenados junto con sus marcas de tiempo. Finalmente, la señal adquirida fue guardada en un archivo CSV (senal_corazon.csv) con dos columnas: tiempo en segundos y voltaje en voltios, lo que permite su posterior análisis o visualización.
```pyton
fs = 250
duration = 60*5
samples = fs * duration
channel = "Dev1/ai0"

data = []
timestamps = []

with nidaqmx.Task() as task:
    task.ai_channels.add_ai_voltage_chan(channel)
    task.timing.cfg_samp_clk_timing(rate=fs, sample_mode=nidaqmx.constants.AcquisitionType.CONTINUOUS)

    print(f"Adquiriendo datos de {channel} a {fs} Hz durante {duration} segundos...")
    task.start()

    start_time = time.time()
    while time.time() - start_time < duration:
        try:
            new_data = task.read(number_of_samples_per_channel=fs, timeout=5)
            data.extend(new_data)
            timestamps.extend(np.linspace(time.time() - start_time, time.time() - start_time, len(new_data)))
        except nidaqmx.errors.DaqError as e:
            print(f" Advertencia: {e}")
            break

    task.stop()

# Guardar los datos en un archivo CSV
filename = "senal_corazon.csv"
with open(filename, mode='w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["Tiempo (s)", "Voltaje (V)"])
    for t, v in zip(timestamps, data):
        writer.writerow([t, v])
```
![image](https://github.com/user-attachments/assets/8f8a4e41-c5e2-4a73-8dd3-912c7392e157)

# 1) calculos del filtro
Se optó por aplicar un filtro digital Butterworth de tipo pasa bajos debido a su característica principal: una respuesta en frecuencia suave, sin ondulaciones en la banda pasante. Esta propiedad resulta especialmente adecuada para mantener la forma original de la señal cardíaca, evitando distorsiones que podrían afectar su análisis.

La frecuencia de corte seleccionada fue de 250 Hz, lo que permite suprimir eficazmente componentes de alta frecuencia, como el ruido generado por la actividad muscular o las interferencias electromagnéticas, sin comprometer la información relevante del ECG.

Por otro lado, se eligió un filtro de orden 5, ya que proporciona una atenuación eficiente fuera de la banda útil sin comprometer la estabilidad numérica del sistema. Evitar órdenes más altos permite reducir el riesgo de inestabilidades o artefactos indeseados, alineándose con los objetivos del presente laboratorio.
```pyton
# Parámetros
fs = 250  # Frecuencia de muestreo en Hz
low = 0.05
high = 40 
orden = 1

# Filtro pasa banda
sos = butter(orden, [low, high], btype='bandpass', fs=fs, output='sos')

# Leer archivo CSV
filename = "senal_corazon.csv"
data = []
timestamps = []

with open(filename, mode='r') as file:
    reader = csv.reader(file)
    next(reader)
    for row in reader:
        timestamps.append(float(row[0]))
        data.append(float(row[1]))

data = np.array(data)

# Filtrar señal
filtered_data = sosfilt(sos, data)
```

# 2) Cálculo de la Frecuencia
Se seleccionó una frecuencia de corte de 250 Hz considerando tanto criterios fisiológicos como prácticos. Este valor permite conservar la información más relevante del ECG, especialmente los componentes de alta frecuencia del complejo QRS, que son fundamentales para la detección precisa de los picos R. Al mismo tiempo, es lo suficientemente bajo como para atenuar el ruido de alta frecuencia que suele estar presente en las señales biomédicas.

Para implementar el filtro digitalmente, esta frecuencia de corte fue normalizada respecto a la frecuencia de Nyquist, que en este caso es de 500 Hz (la mitad de la frecuencia de muestreo, 250 Hz). La normalización es necesaria ya que las funciones de diseño de filtros digitales trabajan en una escala de 0 a 1, donde 1 representa la frecuencia de Nyquist.

Este valor de 0.5 es el que se utiliza internamente en el diseño del filtro para establecer el punto de corte deseado.
# 3) Detección de Picos R
La detección de los picos R se realiza utilizando la función find_peaks() sobre la señal de ECG previamente filtrada. Para asegurar que solo se identifiquen los verdaderos picos R, se establecen criterios específicos como un umbral mínimo de amplitud y una separación mínima entre picos consecutivos. Esta etapa resulta clave, ya que los intervalos entre picos R constituyen la base para el análisis de la variabilidad de la frecuencia cardíaca (HRV), tanto en el dominio temporal como en el análisis mediante transformada wavelet.
```pyton
# Detección de picos R
peaks, _ = find_peaks(normalized_cwt, distance=fs*0.4, height=0.3)
```
# 4) Cálculo de Intervalos R-R
Los intervalos R-R se obtienen calculando la diferencia entre las posiciones de los picos R consecutivos, utilizando np.diff(picos). Posteriormente, estas diferencias se dividen por la frecuencia de muestreo (fs) para expresar el resultado en segundos. Estos intervalos reflejan el tiempo transcurrido entre latidos sucesivos y constituyen la base para el análisis de la variabilidad de la frecuencia cardíaca (HRV).

```pyton
# Calcular intervalos R-R en segundos
rr_intervals = np.diff(np.array(timestamps)[peaks])
```
# 5) Variabilidad de la frecuencia cardiaca 
A partir de los intervalos RR extraídos, se calcularon varias métricas estándar para evaluar la variabilidad de la frecuencia cardíaca (HRV) en el dominio del tiempo. Se obtuvo el valor medio de los intervalos (rr_mean), el cual representa el ritmo cardíaco promedio durante el periodo de análisis.

La desviación estándar (sdnn) se utilizó como medida global de la variabilidad de los intervalos RR, mientras que el valor de rmssd se calculó para evaluar las diferencias cuadráticas medias entre intervalos sucesivos, lo cual refleja la actividad parasimpática a corto plazo.

Asimismo, se computaron las métricas nn50 y pnn50, que cuantifican cuántos pares de intervalos consecutivos difieren en más de 50 ms y qué porcentaje representan del total, respectivamente. Estas últimas también están relacionadas con el control parasimpático del corazón y son útiles para caracterizar cambios rápidos en el ritmo cardíaco.

Este conjunto de indicadores permite una caracterización inicial del comportamiento dinámico de la señal ECG, y sirve como base para la comparación entre sujetos sanos y aquellos con posibles disfunciones autonómicas.


```pyton
# Métricas de HRV
rr_mean = np.mean(rr_intervals)
sdnn = np.std(rr_intervals)
rmssd = np.sqrt(np.mean(np.square(np.diff(rr_intervals))))
nn50 = np.sum(np.abs(np.diff(rr_intervals)) > 0.05)
pnn50 = (nn50 / len(rr_intervals)) * 100 if len(rr_intervals) > 0 else 0
```
# 6) Transformada Wavelet
Se utilizo la Transformada Wavelet Continua (CWT) utilizando la wavelet de Morlet para analizar la señal filtered_data. Se utiliza un rango de escalas (widths) de 1 a 49, y un parámetro w=5 que ajusta la forma de la wavelet Morlet. Posteriormente, se calcula la magnitud absoluta de la transformada (cwt_abs) y se suma a lo largo del eje de escalas para obtener un perfil de energía temporal (cwt_sum).
```pyton
# Transformada Wavelet Continua con Morlet
widths = np.arange(1, 50)
cwt_matrix = cwt(filtered_data, morlet2, widths, w=5)

# Escala de mejor contraste
cwt_abs = np.abs(cwt_matrix)
cwt_sum = np.sum(cwt_abs, axis=0)
normalized_cwt = (cwt_sum - np.min(cwt_sum)) / (np.max(cwt_sum) - np.min(cwt_sum))
```
#Resultados
![image](https://github.com/user-attachments/assets/d3755068-eb7d-4230-9a40-3507b813c110)


## Materiales y Requerimientos
• Python

• matplotlib

• Modulo EKG

• Señal adquirida

## Referencias
•	Addison, P. S. (2005). Wavelet transforms and the ECG: a review. Physiological Measurement, 26(5), R155–R199.

•	Berntson, G. G., Bigger, J. T., Eckberg, D. L., et al. (1997). Heart rate variability: origins, methods, and interpretive caveats. Psychophysiology, 34(6), 623–648.

•	Guyton, A. C., & Hall, J. E. (2011). Textbook of Medical Physiology (12th ed.). Saunders.	

•	Mallat, S. (1999). A Wavelet Tour of Signal Processing. Academic Press.

•	Task Force of the European Society of Cardiology and the North American Society of Pacing and Electrophysiology. (1996). Heart rate variability: standards of measurement, physiological interpretation and clinical use. Circulation, 93(5), 1043–1065.


