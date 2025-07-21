<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Malla Médica Interactiva – Medicina (Médico Cirujano)</title>
  <style>
    body { font-family: 'Segoe UI', Tahoma, sans-serif; background: #f5f7fa; color: #333; padding: 20px; }
    h1 { text-align: center; color: #2a4d69; }
    .malla { display: flex; gap: 10px; overflow-x: auto; }
    .semestre { background: #fff; border-radius: 8px; padding: 10px; min-width: 200px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
    .semestre h2 { font-size: 1.1em; margin-bottom: 8px; color: #4b86b4; }
    .materia { display: block; margin: 4px 0; padding: 6px 8px; border-radius: 4px; cursor: pointer; user-select: none; position: relative; }
    .materia.aprobada { background: #a8e6cf; color: #035e3d; }
    .materia.bloqueada { background: #e0e0e0; color: #777; cursor: not-allowed; }
    .materia.bloqueada:hover::after {
      content: attr(data-tooltip);
      position: absolute; left: 105%; top: 50%;
      transform: translateY(-50%);
      background: #333; color: #fff;
      padding: 4px 8px; border-radius: 4px;
      white-space: nowrap; font-size: 0.9em;
      z-index: 10;
    }
  </style>
</head>
<body>

<h1>Malla Curricular Interactiva – Medicina (Médico Cirujano)</h1>
<div class="malla" id="malla"></div>

<script>
  const estructura = {
    1: ["Anatomía 1","Bioquímica","Histología","Inicio al método científico","Tecnologías para la salud","Autocuidado y conducta saludable"],
    2: [
      { name: "Anatomía 2", req: ["Anatomía 1"] },
      "Fisiología 1","Biología molecular","Genética y embriología","Comunicación médica","Bioestadística"
    ],
    3: [
      "Microbiología y parasitología",
      { name: "Fisiología 2", req: ["Fisiología 1"] },
      "Nutrición","Farmacología","Propedéutica, semiología y diagnóstico físico 1","Epidemiología"
    ],
    4: [
      "Introducción a métodos diagnósticos","Salud pública",
      { name: "Fisiopatología", req: ["Fisiología 2"] },
      "Inmunología básica",
      { name: "Terapéutica farmacológica", req: ["Farmacología"] },
      { name: "Propedéutica, semiología y diagnóstico físico 2", req: ["Propedéutica, semiología y diagnóstico físico 1"] },
      "Técnicas Quirúrgicas"
    ],
    5: ["Cardiología","Neumología","Nefrología","Neurología","Bioética","Atención a la salud 1","Integración clínica 1","Práctica clínica 1"],
    6: [
      "Dermatología","Hematología","Endocrinología","Reumatología",
      { name: "Atención primaria a la salud 2", req: ["Atención a la salud 1"] },
      { name: "Integración clínica 2", req: ["Integración clínica 1"] },
      { name: "Práctica clínica 2", req: ["Práctica clínica 1"] }
    ],
    7: [
      "Cirujía general","Angiología","Urología","Psiquiatría","Gastroenterología","Sistemas y modelos de atención a la salud","Taller médico complementario 1",
      { name: "Integración clínica 3", req: ["Integración clínica 2"] },
      { name: "Práctica clínica 3", req: ["Práctica clínica 2"] }
    ],
    8: [
      "Traumatología y ortopedia","Rehabilitación","Otorrinolaringología","Oftalmología","Oncología","Geriatría","Cuidados paliativos",
      { name: "Taller médico complementario 2", req: ["Taller médico complementario 1"] },
      { name: "Integración clínica 4", req: ["Integración clínica 3"] },
      { name: "Práctica clínica 4", req: ["Práctica clínica 3"] }
    ],
    9: [
      "Ginecología y obstetricia","Urgencias médicas","Calidad en la atención médica","Metodología de la investigación 1",
      { name: "Taller médico complementario 3", req: ["Taller médico complementario 2"] },
      { name: "Integración clínica 5", req: ["Integración clínica 4"] },
      { name: "Práctica clínica 5", req: ["Práctica clínica 4"] }
    ],
    10: [
      "Pediatría","Infectología","Administración en salud",
      { name: "Metodología de la investigación 2", req: ["Metodología de la investigación 1"] },
      { name: "Taller médico complementario 4", req: ["Taller médico complementario 3"] },
      { name: "Integración clínica 4", req: ["Integración clínica 3"] },
      { name: "Práctica clínica 6", req: ["Práctica clínica 5"] }
    ],
    11: ["Internado médico de pregrado 1"],
    12: [
      { name: "Internado médico de pregrado 2", req: ["Internado médico de pregrado 1"] }
    ]
  };

  const estado = JSON.parse(localStorage.getItem("malla_med")) || {};
  function puedeAprobar(materia) {
    return (!materia.req || materia.req.every(r => estado[r]));
  }

  function actualizarYGuardar() {
    localStorage.setItem("malla_med", JSON.stringify(estado));
  }

  function render() {
    const cont = document.getElementById("malla");
    cont.innerHTML = "";
    Object.entries(estructura).forEach(([sem, materias]) => {
      const div = document.createElement("div");
      div.className = "semestre";
      div.innerHTML = `<h2>Semestre ${sem}</h2>`;
      materias.forEach(m => {
        const mat = typeof m === "string" ? { name: m, req: null } : m;
        const span = document.createElement("span");
        span.textContent = mat.name;
        span.className = "materia";
        span.dataset.name = mat.name;
        if (estado[mat.name]) {
          span.classList.add("aprobada");
        } else if (!puedeAprobar(mat)) {
          span.classList.add("bloqueada");
          const faltan = mat.req.filter(r => !estado[r]);
          span.dataset.tooltip = "Falta: " + faltan.join(", ");
        }
        span.addEventListener("click", () => {
          if (span.classList.contains("bloqueada")) return;
          estado[mat.name] = !estado[mat.name];
          actualizarYGuardar();
          render();
        });
        div.appendChild(span);
      });
      cont.appendChild(div);
    });
  }

  render();
</script>

</body>
</html>
