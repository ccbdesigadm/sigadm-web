<template>
    <v-select
      :items="igrejas"
      color="teal darken-3"
      label="Congregação"
      outline
      single-line
      v-model="nome"
      item-text='nome'
      item-value='id'
      append-icon="fas fa-place-of-worship"     
      :rules="[() => !!nome || 'Este campo precisa ser selecionado!']"
    >
      <template slot="item" slot-scope="data">
        <v-flex xs10 sm10>
          <v-list-tile>
            <v-list-tile-content>
              <v-list-tile-title
                @click="SelecionarEncarregadoRegional(data.item.eng_regional),
                SelecionarEncarregadoLocal(data.item.eng_local),
                SelecionarIgreja(data.item.nome)">
                {{data.item.nome}}
              </v-list-tile-title>
            </v-list-tile-content>
          </v-list-tile>
        </v-flex>
        <v-flex xs2 sm2>
          <v-btn flat icon color="red" @click="DeletarNome(data.item.id,data.item.nome)">
            <v-icon>fas fa-trash-alt</v-icon>
          </v-btn>
        </v-flex>
      </template>
    </v-select>
</template>

<script type="text/babel">
import CadNome from "@/pages/Modulos/Patrimonio/CadastroInstrumento/Instrumento/Cadastro/Cadastro_Nome";
import swal from "sweetalert2";

export default {
  name: "Selecao_Igreja_sem",
  components: {
    CadNome
  },
  data() {
    return {
      autorizacao: false,
      igrejas: [{nome: "Central/Natal",eng_regional:"Eutíco", eng_local:"Henrique"}],
      nome: "",
      naipe:"",
      igreja:''
    };
  },
  created() {
    this.AtualizarNomes();
  },

  methods: {
    SelecionarIgreja(igreja) {
      //console.log('Igreja: ',igreja);
      this.$emit("Igreja", { igreja:igreja });      
    },
    SelecionarEncarregadoRegional(encarregado) {
      //console.log('Encarregado: ',encarregado);
      this.$emit("Regional", { encarregado:encarregado });      
    },
    SelecionarEncarregadoLocal(encarregado) {
      //console.log('Encarregado: ',encarregado);
      this.$emit("Local", { encarregado:encarregado });      
    },
    DeletarNome(id,nome) {
      swal
        .fire({
          title: "Deseja excluir o nome " + nome + "?",
          text: "Após a confirmação esta ação não poderá ser revertida!",
          type: "warning",
          showCancelButton: true,
          confirmButtonColor: "#3085d6",
          cancelButtonColor: "#d33",
          confirmButtonText: "Sim, deletar"
        })
        .then(result => {
          if (result.value) {
            this.Deletar(id);
            swal.fire(
              "Deletado!",
              "O nome " + nome + " foi deletado com sucesso.",
              "success"
            );
          }
        });
    },
    Deletar(id) {
      //endpoit que fará a deleção do item nome
      let usuarioAutorize = sessionStorage.getItem("usuario");
      if (!usuarioAutorize) {
        this.$router.push("/");
        const toast = swal.mixin({
          toast: true,
          position: "top-end",
          showConfirmButton: false,
          timer: 3000
        });

        toast({
          type: "error",
          title: "Sem autorização, entre com seus dados novamente!"
        });
      } else if (usuarioAutorize) {
        this.autorizacao = JSON.parse(usuarioAutorize);
        this.$http
          .delete(
            this.$url + "NomeInstrumento/v1/" + id,
            {
              headers: {
                authorization: "Bearer " + this.autorizacao.accessToken
              }
            }
          )
          .then(response => {
            //console.log(response.status);
            //console.log("deletando nome");
            if (response.status == 204) {
              //console.log("dados do nome");
              //console.log(response.data);
              this.AtualizarNomes();
              //Criar um loop pois o dado "data" que é recebido é um array
            } else if (response.status == 401) {
              this.$router.push("/");
              const toast = swal.mixin({
                toast: true,
                position: "top-end",
                showConfirmButton: false,
                timer: 3000
              });
            }
          })
          .catch(e => {
            //console.error(e);
            //console.data(e.data);
            if (e.data.status == 401) {
              this.$router.push("/");
            }
          });
      }
    },
    AtualizarNomes() {
      this.nomes = [];
      let usuarioAutorize = sessionStorage.getItem("usuario");
      this.autorizacao = JSON.parse(usuarioAutorize);
      this.$http
        .get(this.$url + "NomeInstrumento/v1", {
          headers: { authorization: "Bearer " + this.autorizacao.accessToken }
        })
        .then(response => {
          //console.log(response.status);
          if (response.status == 200) {
            //console.log("request ok! ", new Date().toISOString().substr(0, 16));
           this.nomes = response.data
          } else if (response.status == 401) {
                    this.$router.push("/");
          }
        })
        .catch(e => {
          this.$router.push("/");
          //console.error(e);
          //console.data(e.data);
        });
    }
  }
};
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
