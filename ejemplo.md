# Curso Modelaje Metabólico 2026

## 1. **Configuración de COMETS en DockerDesktop**
##### i. Contenedor de COMETS en DockerDesktop
```texto
dukovski/comets-lab:1.0
```
##### ii. Crear imagen
Host port: 8888
##### ii. Instalar git
apt-get update

apt-get install git

##### iii. Clonar repositorio
git clone https://github.com/ecoevolab/ModelajeMetabolico

## 2. **Ejemplo**
##### i. Crecimiento aislado
   a) Simular el crecimiento de solo una bacateria en el tiempo
```texto
!python3 /workspace/ModelajeMetabolico/scr/sim_syncom_comets.py \
--gem_path /workspace/ModelajeMetabolico/modelos \
--strains Escherichia_coli \
--initial_mass 1e-5 \
--cycles 3500 \
--media  m9 \
--outdir ./ecoli
```

  b) ¿Cómo crece la bacteria? Graficar su biomasa en el tiempo.
```texto
import pandas as pd
import matplotlib.pyplot as plt

# Cargar archivo
biomasa = pd.read_csv(
    "/workspace/ecoli/biomass.txt",
    sep=r"\s+",
    header=None
)

# Renombrar columnas
biomasa.columns = [
    "ciclo",
    "col2",
    "col3",
    "modelo",
    "biomasa"
]

# Graficar
plt.figure(figsize=(8, 5))

plt.plot(
    biomasa["ciclo"],
    biomasa["biomasa"],
    linewidth=2,
    color='green'
)

plt.xlabel("Ciclos", fontsize=13)
plt.ylabel("Biomasa (gDW)", fontsize=13)
plt.title("Curva de crecimiento de Rhodococcus erythropolis")

plt.grid(True, alpha=0.3)
plt.tight_layout()

plt.savefig(
    "/workspace/ecoli/curva_crecimiento.png",
    dpi=300,
    bbox_inches="tight"
)

plt.show()
```

  c) ¿Qué metabolitos tienen un flujo metabólico activo? Fitrar metabolítos.

```texto
import pandas as pd

# Cargar datos
flujos = "/workspace/ecoli/Escherichia_coli_exchange_fluxes.tsv"

df_flujos = pd.read_csv(
    flujos,
    sep="\t"
)

# Separar cycle
cycle = df_flujos["cycle"]

# Quedarse solo con columnas numéricas
df_flux = df_flujos.drop(columns=["cycle"])
df_flux = df_flux.apply(pd.to_numeric, errors="coerce")

# Metabolitos con al menos un valor distinto de 0
metabolitos = df_flux.columns[(df_flux != 0).any(axis=0)].tolist()

print(f"{len(metabolitos)} metabolitos con flujo distinto de 0:\n")

for met in metabolitos:
    print(met)
```

   d). ¿Cómo cambia el flujo de EX_glc__D_e (glucosa) ? 
   Graficar el cambio del flujo de EX_glc__D_e en el tiempo.

```text
import matplotlib.pyplot as plt
import pandas as pd

# Cargar datos
flujos = "/workspace/ecoli/Escherichia_coli_exchange_fluxes.tsv"

df_flujos = pd.read_csv(
    flujos,
    sep="\t"
)

# Seleccionar metabolito
metabolito_seleccionado = "EX_glc__D_e"

# Revisar si el metabolito existe
if metabolito_seleccionado not in df_flujos.columns:
    print(f"Error: {metabolito_seleccionado} no existe.")

else:
    plt.figure(figsize=(8, 5))

    plt.plot(
        df_flujos["cycle"],
        df_flujos[metabolito_seleccionado],
        linewidth=2,
        color='orange'
    )

    plt.title(
        f"Flujo de {metabolito_seleccionado}",
        fontweight="bold"
    )

    plt.xlabel("Ciclo")
    plt.ylabel("Flux")

    plt.tight_layout()
    plt.show()
```

 
   e) ¿De qué depende el crecimiento de una bacteria? Comparar cómo cambia la biomasa de una misma bacteria 
   si cambia el medio de cultivo.
```text 
import pandas as pd
import matplotlib.pyplot as plt

# Archivo 1
biomasa1 = pd.read_csv(
    "/workspace/ecoli/biomass.txt",
    sep=r"\s+",
    header=None
)

# Archivo 2
biomasa2 = pd.read_csv(
    "/workspace/ModelajeMetabolico/simulaciones/Ecoli_lb/biomass.txt",
    sep=r"\s+",
    header=None
)

# Renombrar columnas
columnas = [
    "ciclo",
    "col2",
    "col3",
    "modelo",
    "biomasa"
]

biomasa1.columns = columnas
biomasa2.columns = columnas

# Graficar
plt.figure(figsize=(8, 5))

plt.plot(
    biomasa1["ciclo"],
    biomasa1["biomasa"],
    linewidth=2,
    color="green",
    label="medio m9"
)

plt.plot(
    biomasa2["ciclo"],
    biomasa2["biomasa"],
    linewidth=2,
    color='yellow',
    label="medio lb"
)

plt.xlabel("Ciclos", fontsize=13)
plt.ylabel("Biomasa (gDW)", fontsize=13)
plt.title("Comparación de curvas de crecimiento")

plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()

plt.savefig(
    "/workspace/ecoli/comparacion_crecimiento.png",
    dpi=300,
    bbox_inches="tight"
)

plt.show()
```

## 3. Ejercicio

a). ¿Cómo crecen las bacterias cuándo están interactuando? Simulacion bacteria-bacteria.

   * En la carpeta de modelos dentro de `ModelosMetabólicos` encontrarás diferentes modelos metabólicos. Simula cómo crecen juntas *Escherichia_coli* y *Bacillus_subtilis*. 
   Agrega con el argumento --strains el nombre de los modelos metabólicos de ambas.
  
   * Compara cómo crecen ambas bacterias cuando están interactuando, con ayuda del siguiente código, grafica en una sola imagen el crecimiento de ambas, agrega el archivo 
   de biomass.txt con la ruta correspondiente a la carpeta que generaste anteriormente.
   
```text
import pandas as pd
import matplotlib.pyplot as plt

# Cargar archivo
biomasa = pd.read_csv(
    "/workspace/ecoli_vs_bacillus_subtilis/biomass.txt",
    sep=r"\s+",
    header=None
)

# Renombrar columnas
biomasa.columns = [
    "ciclo",
    "x",
    "y",
    "modelo",
    "biomasa"
]

colores = {
    "Escherichia_coli.cmd": "green",
    "Bacillus_subtilis.cmd": "red"
}

# Graficar
plt.figure(figsize=(8, 5))

for modelo in biomasa["modelo"].unique():

    datos = biomasa[
        biomasa["modelo"] == modelo
    ]

    plt.plot(
        datos["ciclo"],
        datos["biomasa"],
        linewidth=2,
        color=colores[modelo],
        label=modelo.replace(".cmd", "")
    )

plt.xlabel("Ciclos", fontsize=13)
plt.ylabel("Biomasa (gDW)", fontsize=13)
plt.title("Curvas de crecimiento")

plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()

plt.show()
```

  * Con ayuda del archivo de flujos metabólicos de E. coli (Escherichia_coli_exchange_fluxes.tsv)
    filtra los metabolitos que tienen flujo metabólico activo.
    
  * ¿Cuántos y cuáles metabolitos tienen flujos metabólicos activos para E. coli?
  
  * Con ayuda de los archivos de flujos metabolico de B. subtilis (Bacillus_subtilis_exchange_fluxes.tsv
  filtra los metabolitos que tienen flujo metabólico activo.
  
  * ¿Cuántos y cuáles metabolitos tienen flujos metabólicos activos para E. coli?

  * ¿Qué metabolitos tienen flujos metabólicos activos solo en E. coli? 
  
  * ¿Qué metabolitos tienen flujos metabólicos activos solo en B. subtilis? 
  
  * ¿Qué metabolitos tiene flujos metabólicos activos en ambas bacterias?
  
  * Grafica cómo cambia el consumo de EX_co2_e (dióxido de carbono) en E. coli.
  
  * Grafica cómo cambia el consumo de EX_co2_e (dióxido de carbono) en B. subtiles.












