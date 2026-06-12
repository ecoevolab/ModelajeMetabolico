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
   i. Simular el crecimiento de solo una bacateria en el tiempo
```texto
!python3 /workspace/ModelajeMetabolico/scr/sim_syncom_comets.py \
--gem_path /workspace/ModelajeMetabolico/modelos \
--strains Escherichia_coli \
--initial_mass 1e-5 \
--cycles 3500 \
--media  m9 \
--outdir ./ecoli
```

  ii. ¿Cómo crece que bacteria? Graficar su biomasa. Graficar su biomasa en el tiempo
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

  iii. ¿Qué metabolitos tienen un flujo? Fitrar metabolicos cuyos flujos cambien en el tiempo

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

   iv. ¿Cómo cambia el flujo EX_glc__D_e (glucosa) ? 
   Graficar como cambia el flujo de EX_glc__D_e en el tiempo

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

 
   v. ¿De que depende el crecimiento de una bacteria? Comparar como cambia la misma bacteria si cambia el medio de cultivo.
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

i. ¿Cómo crecen las bacterias cuándo están interactuando? Simulacion bacteria-bacteria.

   a. En la carpeta de modelos dentro de `ModelosMetabólicos` encontrarás diferentes modelos metabólicos. 
   
   Simula cómo crecen juntas *Escherichia coli* y *Bacillus_subtilis*. Agrega en con el argumento 
   --strains el nombre de los modelos metabolicos de ambas
```text
!python3 /workspace/ModelajeMetabolico/scr/sim_syncom_comets.py \
--gem_path /workspace/ModelajeMetabolico/modelos \
--strains Escherichia_coli Bacillus_subtilis\
--initial_mass 1e-5 \
--cycles 3500 \
--media  m9 \
--outdir ./ecoli_vs_bsubtilis
```
  
  
  b. Compara como crecen ambas bacterias cuando están interactuando, con ayuda del siguiente codigo, grafica en una sola imagen el crecimiento de ambas.
```text
import pandas as pd
import matplotlib.pyplot as plt

# Cargar archivo
biomasa = pd.read_csv(
    "/workspace/ecoli_vs_bsubtilis/biomass.txt",
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

ii. ¿Qué está consumiendo cada bacteria? 

  a. con ayuda de los archivos de flujos metabolico (Bacillus_subtilis_exchange_fluxes.tsv y Escherichia_coli_exchange_fluxes)
    filtra los metabolicos que cada una esta consumiendo.
    
```text
import pandas as pd

# Cargar datos
flujos = "/workspace/ecoli_vs_bsubtilis/Escherichia_coli_exchange_fluxes.tsv"

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

```text
import pandas as pd

# Cargar datos
flujos = "/workspace/ecoli_vs_bsubtilis/Bacillus_subtilis_exchange_fluxes.tsv"

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
  b. ¿cuantos y cuales metabolitos tienen flujos metabolitos activos para cada bacteria?
  Ecoli: 24 metabolitos con flujo distinto de 0:

EX_4hba_e
EX_4hbald_e
EX_ca2_e
EX_cl_e
EX_co2_e
EX_cobalt2_e
EX_cu2_e
EX_etoh_e
EX_fe2_e
EX_fe3_e
EX_for_e
EX_glc__D_e
EX_h2o_e
EX_h_e
EX_k_e
EX_mg2_e
EX_mn2_e
EX_na1_e
EX_nh4_e
EX_o2_e
EX_pi_e
EX_so4_e
EX_val__L_e
EX_zn2_e

B. subtilis: 
29 metabolitos con flujo distinto de 0:

EX_4hba_e
EX_4hbald_e
EX_R_3hcmrs7e_e
EX_R_3hpt_e
EX_ca2_e
EX_cl_e
EX_co2_e
EX_cobalt2_e
EX_cu2_e
EX_etoh_e
EX_fe2_e
EX_fe3_e
EX_for_e
EX_glc__D_e
EX_h2o_e
EX_h_e
EX_hco3_e
EX_k_e
EX_lac__L_e
EX_mg2_e
EX_mn2_e
EX_na1_e
EX_nh4_e
EX_o2_e
EX_pi_e
EX_ppap_e
EX_so4_e
EX_val__L_e
EX_zn2_e

  ¿que metabolitos tienen flujos metabolicos activos unicos en cada bacteria? 
  E. coli: EX_zn2_e
  B. subtilis: EX_R_3hcmrs7e_e
EX_R_3hpt_e
EX_hco3_e
EX_lac__L_e
EX_ppap_e

  ¿que metabolitos se encoentran con flujos metabolicos activos en ambas bacterias bacteria?
  EX_4hba_e
EX_4hbald_e
EX_ca2_e
EX_cl_e
EX_co2_e
EX_cobalt2_e
EX_cu2_e
EX_etoh_e
EX_fe2_e
EX_fe3_e
EX_for_e
EX_glc__D_e
EX_h2o_e
EX_h_e
EX_k_e
EX_mg2_e
EX_mn2_e
EX_na1_e
EX_nh4_e
EX_o2_e
EX_pi_e
EX_so4_e
EX_val__L_e
  b. Para cada bacteria grafica el metabolito: EX_co2_e 
```text
# Cargar datos
flujos = "/workspace/ecoli_vs_bsubtilis/Escherichia_coli_exchange_fluxes.tsv"

df_flujos = pd.read_csv(
    flujos,
    sep="\t"
)

# Seleccionar metabolito
metabolito_seleccionado = "EX_co2_e"

# Revisar si el metabolito existe
if metabolito_seleccionado not in df_flujos.columns:
    print(f"Error: {metabolito_seleccionado} no existe.")

else:
    plt.figure(figsize=(8, 5))

    plt.plot(
        df_flujos["cycle"],
        df_flujos[metabolito_seleccionado],
        linewidth=2.5
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

```text
import matplotlib.pyplot as plt
import pandas as pd

# Cargar datos
flujos = "/workspace/ecoli_vs_bsubtilis/Bacillus_subtilis_exchange_fluxes.tsv"

df_flujos = pd.read_csv(
    flujos,
    sep="\t"
)

# Seleccionar metabolito
metabolito_seleccionado = "EX_co2_e"

# Revisar si el metabolito existe
if metabolito_seleccionado not in df_flujos.columns:
    print(f"Error: {metabolito_seleccionado} no existe.")

else:
    plt.figure(figsize=(8, 5))

    plt.plot(
        df_flujos["cycle"],
        df_flujos[metabolito_seleccionado],
        linewidth=2.5
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
  
    
    
  
    












