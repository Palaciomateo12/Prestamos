<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Panel de Administración - Préstamos</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- SDK oficial de Supabase -->
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
</head>
<body class="bg-slate-100 text-slate-800 font-sans min-h-screen p-6">

  <div class="max-w-6xl mx-auto space-y-6">
    <div class="flex justify-between items-center bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
      <div>
        <h1 class="text-2xl font-bold text-slate-900">Panel de Control General</h1>
        <p class="text-xs text-slate-500">Solicitudes registradas en tiempo real desde Supabase</p>
      </div>
      <button onclick="cargarPrestamos()" class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold text-xs px-4 py-2.5 rounded-xl shadow transition-all">
        Actualizar Tabla
      </button>
    </div>

    <!-- TABLA DE SOLICITUDES -->
    <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
      <table class="w-full text-left text-sm text-slate-600">
        <thead class="bg-slate-50 text-slate-500 text-xs uppercase border-b border-slate-200">
          <tr>
            <th class="p-4">Cliente / DNI</th>
            <th class="p-4">Contacto</th>
            <th class="p-4">Monto / Devuelto</th>
            <th class="p-4">Cuotas</th>
            <th class="p-4">Estado</th>
            <th class="p-4 text-center">Acciones</th>
          </tr>
        </thead>
        <tbody id="tablaCuerpo" class="divide-y divide-slate-100">
          <tr>
            <td colspan="6" class="p-4 text-center text-slate-400">Cargando datos...</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>

  <script>
    // CONFIGURACIÓN DE SUPABASE
    const SUPABASE_URL = 'https://ndetpdwwiemjgqarnvoi.supabase.co';
    const SUPABASE_ANON_KEY = sb_publishable_oq80v6WGHQ1Vh38NffoXQA_W7J4dB7S';
    const supabase = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

    async function cargarPrestamos() {
      const cuerpo = document.getElementById('tablaCuerpo');
      cuerpo.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-slate-400">Consultando servidor...</td></tr>';

      // Obtener lista desde Supabase ordenado por fecha
      const { data, error } = await supabase
        .from('prestamos')
        .select('*')
        .order('id', { ascending: false });

      if (error) {
        cuerpo.innerHTML = `<tr><td colspan="6" class="p-4 text-center text-red-500">Error: ${error.message}</td></tr>`;
        return;
      }

      if (data.length === 0) {
        cuerpo.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-slate-400">No hay solicitudes registradas aún.</td></tr>';
        return;
      }

      cuerpo.innerHTML = '';
      data.forEach(p => {
        cuerpo.innerHTML += `
          <tr class="hover:bg-slate-50">
            <td class="p-4 font-semibold text-slate-900">
              DNI: ${p.dni}<br>
              <span class="text-xs text-slate-400 font-normal">${p.email}</span>
            </td>
            <td class="p-4 text-xs">${p.tel}</td>
            <td class="p-4">
              <span class="font-bold text-slate-800">$${p.monto.toLocaleString('es-AR')}</span><br>
              <span class="text-xs text-emerald-600">Devuelto: $${p.monto_devuelto.toLocaleString('es-AR')}</span>
            </td>
            <td class="p-4 text-xs font-semibold">${p.cuotas_pagadas} / ${p.cuotas_totales}</td>
            <td class="p-4">
              <span class="px-2.5 py-1 text-xs font-bold rounded-full ${p.estado === 'Activo' ? 'bg-emerald-100 text-emerald-800' : 'bg-amber-100 text-amber-800'}">
                ${p.estado}
              </span>
            </td>
            <td class="p-4 text-center space-x-2">
              <button onclick="cambiarEstado(${p.id}, 'Activo')" class="bg-emerald-500 text-white text-xs px-2.5 py-1 rounded hover:bg-emerald-600">Aprobar</button>
              <button onclick="cambiarEstado(${p.id}, 'Rechazado')" class="bg-rose-500 text-white text-xs px-2.5 py-1 rounded hover:bg-rose-600">Rechazar</button>
            </td>
          </tr>
        `;
      });
    }

    async function cambiarEstado(id, nuevoEstado) {
      const { error } = await supabase
        .from('prestamos')
        .update({ estado: nuevoEstado })
        .eq('id', id);

      if (error) {
        alert('No se pudo actualizar el estado: ' + error.message);
      } else {
        cargarPrestamos();
      }
    }

    // Carga inicial al abrir la página
    cargarPrestamos();
  </script>
</body>
</html>
