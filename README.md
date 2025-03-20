//////////////////////////////////////////////////////////////////////////////////////////////////////////
//                                                                                       _              //
//               _    _       _      _        _     _   _   _    _   _   _        _   _  _   _          //
//           |  | |  |_| |\| |_| |\ |_|   |\ |_|   |_| |_| | |  |   |_| |_| |\/| |_| |  |_| | |   /|    //    
//         |_|  |_|  |\  | | | | |/ | |   |/ | |   |   |\  |_|  |_| |\  | | |  | | | |_ | | |_|   _|_   //
//                                                                                       /              //
//////////////////////////////////////////////////////////////////////////////////////////////////////////

/*
*   Programa básico para controle da placa durante a Jornada da Programação 1
*   Permite o controle das entradas e saídas digitais, entradas analógicas, display LCD e teclado. 
*   Cada biblioteca pode ser consultada na respectiva pasta em componentes
*   Existem algumas imagens e outros documentos na pasta Recursos
*   O código principal pode ser escrito a partir da linha 86
*/

// Área de inclusão das bibliotecas
//-----------------------------------------------------------------------------------------------------------------------
/*#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"

#include "HCF_IOTEC.h"   // Vai se tornar HCF_IOTEC
#include "HCF_LCD.h" // Vai se tornar HCF_LCD
#include "HCF_ADC.h"   // Vai se tornar HCF_ADC
#include "HCF_MP.h"   // Vai se tornar HCF_MP
// Incluir HCF_IOT HCF_BT HCF_DHT HCF_ULTRA HCF_RFID HCF_ZMPT HCF_ACS HCF_SERVO HCF_OLED HCF_CAM HCF_SD HCF_LORA


// Área das macros
//-----------------------------------------------------------------------------------------------------------------------

#define IN(x) (entradas>>x)&1

// Área de declaração de variáveis e protótipos de funções
//-----------------------------------------------------------------------------------------------------------------------

static const char *TAG = "Placa";
static uint8_t entradas, saidas = 0; //variáveis de controle de entradas e saídas
static char tecla = '-' ;
char escrever[40];

// Funções auxiliares
//-----------------------------------------------------------------------------------------------------------------------

void abrir_porta(void) {
    ESP_LOGI(TAG, "Abrindo porta...");
    saidas |= (1 << 7); // Sinal de abertura na saída 7
    io_le_escreve(saidas);
    vTaskDelay(pdMS_TO_TICKS(500)); // Pulso de 0,5s
    saidas &= ~(1 << 7);
    io_le_escreve(saidas);
    ESP_LOGI(TAG, "Porta fechada.");
}

// Programa Principal
//-----------------------------------------------------------------------------------------------------------------------

void app_main(void)
{
    /////////////////////////////////////////////////////////////////////////////////////   Programa principal
    escrever[39] = '\0';

    // a seguir, apenas informações de console, aquelas notas verdes no início da execução
    ESP_LOGI(TAG, "Iniciando...");
    ESP_LOGI(TAG, "Versão do IDF: %s", esp_get_idf_version());

    /////////////////////////////////////////////////////////////////////////////////////   Inicializações de periféricos (manter assim)
    
    // inicializar os IOs e teclado da placa
    iniciar_iotec();      
    entradas = io_le_escreve(saidas); // Limpa as saídas e lê o estado das entradas

    // inicializar o display LCD 
    iniciar_lcd();
    escreve_lcd(1,0,"Apertar tecla =");
    escreve_lcd(2,0," Programa Basico");
    
    // Inicializar o componente de leitura de entrada analógica
    esp_err_t init_result = iniciar_adc_CHX(0);
    if (init_result != ESP_OK) {
        ESP_LOGE(TAG, "Erro ao inicializar o componente ADC personalizado");
    }

    //delay inicial
    vTaskDelay(1000 / portTICK_PERIOD_MS); 
    limpar_lcd();

    /////////////////////////////////////////////////////////////////////////////////////   Periféricos inicializados

    /////////////////////////////////////////////////////////////////////////////////////   Início do ramo principal                    
    while (1)                                                                                                                         
    {                                                                                                                                 
        tecla = le_teclado();     
        if(tecla > '0' && tecla <= '8')
        {
            saidas = 1 << (tecla - '0' - 1);
        }            
        else if (tecla == '9')
        {
            saidas = 0xFF;
        }                                  
        else if (tecla == '0')
        {
            saidas = 0x00;
        }    
        else if (tecla == '=')
        {
            abrir_porta();
        }                                                                                                   
        
        entradas = io_le_escreve(saidas);

        sprintf(escrever, "INPUTS:%d%d%d%d%d%d%d%d", IN(7),IN(6),IN(5),IN(4),IN(3),IN(2),IN(1),IN(0));
        escreve_lcd(1,0,escrever);

        sprintf(escrever, "%c", tecla);
        escreve_lcd(2,14,escrever);
        
        vTaskDelay(10 / portTICK_PERIOD_MS);    // delay mínimo obrigatório, se retirar, pode causar reset do ESP
    }
    
    // caso erro no programa, desliga o módulo ADC
    adc_limpar();

    /////////////////////////////////////////////////////////////////////////////////////   Fim do ramo principal
    
}
*/
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"
#include <string.h>

#include "HCF_IOTEC.h"
#include "HCF_LCD.h"
#include "HCF_ADC.h"
#include "HCF_MP.h"

// Área das macros
//-----------------------------------------------------------------------------------------------------------------------

#define IN(x) (entradas>>x)&1

// Área de declaração de variáveis e protótipos de funções
//-----------------------------------------------------------------------------------------------------------------------

static const char *TAG = "Placa";
static uint8_t entradas, saidas = 0;
static char tecla = '-';
char escrever[40];

static char senha_digitada[5] = "";  // Senha armazenada com espaço para '\0'
static int tentativas = 0;
static const char senha_correta[5] = "1234";  

// Funções auxiliares
//-----------------------------------------------------------------------------------------------------------------------

void abrir_porta(void) {
    ESP_LOGI(TAG, "Abrindo porta...");
    saidas |= (1);
    io_le_escreve(saidas);
    vTaskDelay(pdMS_TO_TICKS(500));
    saidas &= ~(1);
    io_le_escreve(saidas);
    ESP_LOGI(TAG, "Porta fechada.");
}

void resetar_senha(void) {
    memset(senha_digitada, 0, sizeof(senha_digitada));  // Reset seguro
}

// Programa Principal
//-----------------------------------------------------------------------------------------------------------------------

void app_main(void)
{
    escrever[39] = '\0';

    ESP_LOGI(TAG, "Iniciando...");
    ESP_LOGI(TAG, "Versão do IDF: %s", esp_get_idf_version());

    iniciar_iotec();      
    entradas = io_le_escreve(saidas);

    iniciar_lcd();
    escreve_lcd(1, 0, "Apertar tecla =");
    escreve_lcd(2, 0, " Programa Basico");
    
    esp_err_t init_result = iniciar_adc_CHX(0);
    if (init_result != ESP_OK) {
        ESP_LOGE(TAG, "Erro ao inicializar o ADC");
    }

    vTaskDelay(1000 / portTICK_PERIOD_MS); 

    while (1) {                                                                                                                          
        tecla = le_teclado();  
        
        if (tecla >= '0' && tecla <= '9') {
            int len = strlen(senha_digitada);
            if (len < 4) {
                senha_digitada[len] = tecla;
                senha_digitada[len + 1] = '\0';  // Assegura terminação correta
            }
        } 
        else if (tecla == '=') {  
            if (strcmp(senha_digitada, senha_correta) == 0) {
                abrir_porta();
                tentativas = 0;  
            } else {
                ESP_LOGI(TAG, "Senha incorreta.");
                resetar_senha();
                tentativas++; 
            }
        } 
        else if (tecla == 'C') {  
            resetar_senha();
        }

        // Exibe a senha mascarada com '*'
        int len = strlen(senha_digitada);
        sprintf(escrever, "Senha: %.*s", len, "****");
        escreve_lcd(2, 0, escrever); 

        // Exibe o número de tentativas
        sprintf(escrever, "Tentativas: %d", tentativas);
        escreve_lcd(1, 0, escrever);

        vTaskDelay(10 / portTICK_PERIOD_MS);
    }
    
    adc_limpar();
}
