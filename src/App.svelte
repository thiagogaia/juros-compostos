<script lang="ts">
  import { imask } from '@imask/svelte';
  // import type Number from '@imask/svelte';
  let aporteInicial_ = '';
  let aporteMensal_ = '';
  let jurosMes_ = '';
  let periodo_ = 1;
  let periodoEm_ = 'anos';
  function formatarMoeda(value) {
      return `R$ ${new Intl.NumberFormat('pt-BR').format(value.toFixed(2))}`;
  }

  const mascararCampoMoeda = {
    mask: 'num',
    blocks: {
      num: {
        mask: Number,
        scale: 2,
        signed: true,
        thousandsSeparator: '',
        padFractionalZeros: true,
        normalizeZeros: true,
        radix: ',',
        mapToRadix: ['.'],
      }
    }
  }

  function moedaParaFloat(valor) {
    if (!valor) {
        return 0;
    }
    valor = valor.split('.').join('');
    valor = valor.split(',').join('.');
    return Number(valor.replace(/[^0-9.]/g, ""));
  }

  function calculoImpostoRenda(periodo) {
    if (periodo < 6) {
        return 0.225
    } else if (periodo >= 6 && periodo < 12) {
        return 0.20
    } else if (periodo >= 12 && periodo < 24) {
        return 0.175
    } else if (periodo >= 24) {
        return 0.15
    } else {
        throw new Error('Erro em calculoImpostoRenda para ' + periodo)
    }
  }

function calcular() {

  const aporteInicial = moedaParaFloat(aporteInicial_);
  const aporteMensal = moedaParaFloat(aporteMensal_);
  var jurosMes = moedaParaFloat(jurosMes_);
  var periodo = periodo_;
  const periodoEm = periodoEm_;
  console.log(jurosMes, periodo, periodoEm, aporteInicial)
  
  if (!jurosMes) {
    alert("Preencha a taxa de juros");
    return false;
  }

  jurosMes = jurosMes / 100;
  periodo = periodoEm == "anos" ? periodo * 12 : periodo;

  const valorFuturo = (aporteInicial * ((1 + jurosMes) ** periodo)) +
    aporteMensal * (
        ((1 + jurosMes) ** periodo) - 1
    ) / jurosMes;

  const totalAplicado = (aporteMensal * periodo) + aporteInicial;

  const valorGanhoBruto = valorFuturo - totalAplicado;

  const imposto = calculoImpostoRenda(periodo);
  const valorGanhoLiquido = valorGanhoBruto - (valorGanhoBruto * imposto);

  document.getElementById('valorGanhoBruto').innerText = formatarMoeda(valorGanhoBruto)
  document.getElementById('valorGanhoLiquido').innerText = formatarMoeda(valorGanhoLiquido)
  document.getElementById('imposto').innerText = `${(imposto * 100)}% <-> ${formatarMoeda(imposto * valorGanhoBruto)}`

  document.getElementById('totalBruto').innerText = formatarMoeda(valorFuturo)
  document.getElementById('totalAplicado').innerText = formatarMoeda(totalAplicado)

  document.getElementById('totalLiquido').innerText = formatarMoeda(totalAplicado + valorGanhoLiquido)

  document.getElementsByClassName('resultado')[0].scrollIntoView({
    behavior: 'smooth'
  });
}
</script>


<div class="calculadora">
  <form>
    <figure class="text-center">
      <h4>CALCULADORA DE RENDA FIXA</h4>
      <figcaption class="blockquote-footer">
          Faça simulações de renda fixa
      </figcaption>
    </figure>
      <div class="form-icon">
        <i class="fas fa-calculator"></i>
      </div>
      <div class="form-row">
        <div class="form-group col-md-6">
          <label for="inputAporteInicial">Investimento inicial</label>
          <div class="input-group">
            <div class="input-group-prepend">
              <span class="input-group-text">R$</span>
            </div>
            <input 
              type="text" 
              class="form-control" 
              id="aporte-inicial" 
              bind:value={aporteInicial_} 
              use:imask={mascararCampoMoeda}
              placeholder="0,00">
          </div>
        </div>
          <div class="form-group col-md-6">
            <label for="inputAporteMensal">Investimento mensal</label>
            <div class="input-group">
              <div class="input-group-prepend">
                <span class="input-group-text">R$</span>
              </div>
              <input 
                type="text" 
                class="form-control" 
                id="aporte-mensal"
                bind:value={aporteMensal_} 
                use:imask={mascararCampoMoeda}
                placeholder="0,00">
            </div>
          </div>
      </div>
      <div class="form-row">
        <div class="form-group col-md-6">
          <label for="inputJurosMes">Taxa de juros ao mês</label>
          <div class="input-group">
            <div class="input-group-prepend">
              <span class="input-group-text">%</span>
            </div>
            <input 
              type="text" 
              class="form-control" 
              id="juros-mes" 
              placeholder="0,00"
              bind:value={jurosMes_} 
              use:imask={mascararCampoMoeda}
              >
          </div>
        </div>
          <div class="form-group col-md-6">
            <label for="inputPeriodoEm">Período em</label>
            <div class="input-group">
              <div class="input-group-prepend">
                <span class="input-group-text"><i class="fas fa-clock"></i></span>
              </div>
              <input type="number" class="form-control" id="periodo" bind:value={periodo_}>
              <div class="input-group-append">
                <select class="custom-select" id="periodo-em" bind:value={periodoEm_}>
                  <option value="anos" selected>anos</option>
                  <option value="meses">meses</option>
                </select>
              </div>
            </div>
          </div>
      </div>
      <div class="form-group">
        <button class="btn btn-block btn-calcular" on:click|preventDefault={calcular}>Calcular</button>
      </div>
  </form>
  {aporteInicial_}
  {aporteMensal_}
  <div class="resultado">
      <h5>Resultado</h5>
      <div class="table-responsive">
          <table class="table">
              <thead>
                  <th>DESCRIÇÃO</th>
                  <th>VALOR</th>
              </thead>
              <tbody>
                  <tr>
                      <td scope="row">Total aplicado</td>
                      <td id="totalAplicado" class="text-info"></td>
                  </tr>
                  <tr>
                      <td scope="row">Valor ganho em juros</td>
                      <td id="valorGanhoBruto" class="text-success"></td>
                  </tr>
                  <tr>
                      <td scope="row">Total de rendimento líquido</td>
                      <td id="valorGanhoLiquido" class="text-success"></td>
                  </tr>
                  <tr>
                      <td scope="row">Total bruto</td>
                      <td id="totalBruto" class="text-success"></td>
                  </tr>
                  <tr>
                      <td scope="row">Total líquido a receber</td>
                      <td id="totalLiquido" class="text-success"></td>
                  </tr>
                  <tr>
                      <td scope="row">Imposto de renda sobre a rentabilidade</td>
                      <td id="imposto" class="text-danger"></td>
                  </tr>
              </tbody>
          </table>
      </div>
  </div>
</div>

<style>
  .calculadora {
    padding: 50px 0;
}

.calculadora form {
    background-color: #fff;
    max-width: 850px;
    margin: auto;
    padding: 50px 70px;
    border-radius: 30px;
    box-shadow: 0px 2px 10px rgb(0 0 0 / 8%);
}

.calculadora .resultado {
    max-width: 850px;
    background-color: #fff;
    margin: auto;
    margin-top: 50px;
    padding: 45px;
    color: #495057;
    border-radius: 30px;
    box-shadow: 0px 2px 10px rgb(0 0 0 / 8%);
}

.calculadora .form-icon {
    text-align: center;
    background-color: #5891ff;
    border-radius: 50%;
    font-size: 40px;
    color: white;
    width: 100px;
    height: 100px;
    margin: auto;
    margin-bottom: 50px;
    line-height: 100px;
}

.calculadora .btn-calcular {
    border-radius: 30px;
    padding: 10px 20px;
    font-size: 18px;
    font-weight: bold;
    background-color: #5791ff;
    border: none;
    color: white;
    margin-top: 20px;
}

@media (max-width: 576px) {
    .calculadora form {
        padding: 50px 20px;
    }

    .calculadora .form-icon {
        width: 70px;
        height: 70px;
        font-size: 30px;
        line-height: 70px;
    }
}
</style>