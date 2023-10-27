# PyBallistics
Biblioteca Python para resolver o OZVB em formulações termodinâmicas e gasodinâmicas.

## Instalação
Você pode instalar a biblioteca através do gerenciador de pacotes [pip](https://en.wikipedia.org/wiki/Pip_(package_manager)). Para fazer isso, basta executar o comando no console

```
pip install --upgrade pyballistics
```

## Breves instruções de uso

Um exemplo de cálculo do OZVB na formulação termodinâmica do problema AGARD:

```python
from pyballistics import ozvb_termo, get_options_agard

opts = get_options_agard() # obtenha um dicionário com os dados iniciais da tarefa AGARD
result = ozvb_termo(opts)  # faça cálculos e obtenha o resultado
```
A variável ```result``` é um [dicionário](https://pythonworld.ru/tipy-dannyx-v-python/slovari-dict-funkcii-i-metody-slovarej.html), que contém o cálculo resultados.

```python
import matplotlib.pyplot as plt # se não houver biblioteca matplotlib, você pode instalá-la usando o comando pip install matplotlib

plt.plot(result['t'], result['p_m']) # pressão balística média versus tempo
plt.grid()  # grade no gráfico
plt.show()  # mostra gráfico
```

Como resultado, será obtido o seguinte gráfico da dependência da pressão balística média (Pa) em relação ao tempo (s):
![](imgs/1.png)

O dicionário ```result``` contém todos os dados necessários para análise posterior:

```python
import numpy as np 

# pressão máxima
print(np.max(result['p_m']))
>>> 319103989.57

# velocidade inicial
print(result['v_p'][-1])
>>> 671.16

# parcela de pólvora queimada
print(result['psi_1'][-1])
>>> 0.932
```

## Instruções mais detalhadas

Duas funções são responsáveis ​​pelos cálculos do OZVB: ```ozvb_termo``` e ```ozvb_lagrange```. Eles podem ser importados diretamente da biblioteca:

```python
from pyballistics import ozvb_termo, ozvb_lagrange
```
A função ```ozvb_termo``` realiza cálculos termodinâmicos, e ```ozvb_lagrange``` realiza cálculos gasodinâmicos em coordenadas Lagrangianas.

Ambas as funções tomam como entrada dicionários, que armazenam todos os dados iniciais necessários para o cálculo. Exemplos de tais dicionários podem ser obtidos nas funções ```get_options_agard``` e ```get_options_sample```:

```python
from pyballistics import get_options_agard, get_options_sample

opts1 = get_options_agard()
opts2 = get_options_sample()

print(opts2)
# {
#   'powders': [
#       {'omega': 7, 'dbname': 'ДГ-4 15/1'},
#       {'omega': 6, 'dbname': '22/7'}],
#  'init_conditions': {
#       'q': 51.76,
#       'd': 0.122,
#       'W_0': 0.0325,
#       'phi_1': 1.02,
#       'p_0': 30000000.0 },
#  'igniter': {
#       'p_ign_0': 1000000.0},
#  'meta_termo': {
#       'dt': 5e-06, 
#       'method': 'rk2'},
#  'meta_lagrange': {
#       'CFL': 0.9, 
#       'n_cells': 150},
#  'stop_conditions': {
#       'v_p': 690, 
#       'p_max': 600000000.0, 
#       'x_p': 9}
# }
```
Um dicionário com dados de entrada é uma estrutura de dados hierárquica (dicionário de dicionários, listas, etc.). Para indicar todos os dados que descrevem de forma inequívoca a tarefa da gestão da segurança e saúde no trabalho, o dicionário deve ser bastante complicado e inconveniente de formar. Para simplificar a formação do dicionário, muitos dos elementos nele incluídos possuem valores padrão e, se necessário, podem ser omitidos. No entanto, existem vários elementos e valores que devem ser especificados.
O dicionário de entrada consiste nos seguintes elementos de “nível superior”:
 - ```'init_conditions'``` - **seção obrigatória**. Esta seção armazena um dicionário com dados iniciais:
    - ```'q'``` - massa do projétil em kg.
    - ```'d'``` - calibre, m.
    - ```'W_0'``` - volume inicial da câmara, m^3.
    - ```'phi_1'``` - coeficiente que leva em consideração a força de atrito no rifle (participa da fórmula de cálculo do coeficiente de massa fictícia do projétil).
    - ```'p_0'``` - pressão de reforço, Pa.
    - ```'T_0'```(*opcional*) - temperatura inicial, K (*valor padrão 293,15 K*).
    - ```'n_S'```(*opcional*) - coeficiente para levar em consideração a área de estrias (*valor padrão 1). A área da seção transversal é calculada usando a fórmula S = n_S pi d^2/4.
    

 - ```'powders'``` - **seção obrigatória**. Esta seção contém uma [lista](https://pythonworld.ru/tipy-dannyx-v-python/spiski-list-funkcii-i-metody-spiskov.html) com dados sobre as amostras de pólvora que compõem a carga do propelente . **Deve ter pelo menos um elemento**. Cada elemento da lista é responsável por seu próprio link e também é um dicionário com os seguintes elementos:
    - ```'omega'``` - peso da carga, kg.
    - ```'dbname'```(*opcional*) - o nome da pólvora no banco de dados. **Se especificado, serão definidos valores padrão para os demais elementos. Se não for especificado, então todos os outros elementos precisarão ser inicializados** (a lista de nomes disponíveis pode ser obtida na função ```get_powder_names```). Aqueles. Se desejar, você pode ajustar algumas características dos pós tabulares e tomar o restante dos valores como padrão.
    - ```'I_e'```(*opcional se ```'dbname'```* for especificado) - impulso de fim de combustão, Pa s.
    - ```'nu'``` (*opcional se ```'dbname'```* for especificado) - expoente na lei de potência de combustão (*padrão 1*).
    - ```'b'```(*opcional se ```'dbname'```* for especificado) - covolume de gases em pó, m^3/kg.
    - ```'delta'```(*opcional se ```'dbname'```* for especificado) - densidade do pó, kg/m^3.
    - ```'f'```(*opcional se ```'dbname'```* for especificado) - força do pó, J/kg.
    - ```'k'```(*opcional se ```'dbname'```* for especificado) - coeficiente adiabático de gases em pó.
    - ```'T_p'```(*opcional se ```'dbname'```* for especificado) - temp. queima de pólvora, K.
    - ```'z_e'```(*opcional se ```'dbname'```* for especificado) - a espessura relativa da camada queimada no final da combustão.
    - ```'kappa_1'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'lambda_1'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'mu_1'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'kappa_2'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'lambda_2'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'mu_2'```(*opcional se ```'dbname'```* for especificado) - coeficiente na lei geométrica da combustão.
    - ```'k_I'```(*opcional se ```'dbname'```* for especificado) - coeficiente de recálculo do impulso de fim de combustão para outras temperaturas iniciais, 1/K.
    - ```'k_f'```(*opcional se ```'dbname'```* for especificado) - coeficiente para recálculo da força do pó para outras temperaturas iniciais, 1/K.
- ```'igniter'``` - **seção obrigatória**. Esta seção armazena um dicionário com dados iniciais relacionados ao dispositivo de ignição. O dicionário possui os seguintes elementos:
    - ```'p_ign_0'``` - pressão do flash, Pa.
    - ```'k_ign'```(*opcional*) - coeficiente adiabático dos gases de ignição (*valor padrão 1,22*).
    - ```'T_ign'```(*opcional*) - temperatura de combustão do ignitor, K (*valor padrão 2427*).
    - ```'f_ign'```(*opcional*) - força do dispositivo de ignição, J/kg (*valor padrão 260.000 J/kg*).
    - ```'b_ign'```(*opcional*) - covolume de gases de ignição, m^3/kg (*valor padrão 0,0006*).
  - ```'windage'``` - **seção opcional**. Esta seção armazena um dicionário com dados iniciais relacionados à força de resistência do ar na frente do projétil. Se você não especificar este elemento, os valores padrão serão usados. O dicionário possui os seguintes elementos:
    - ```'shock_wave'```(*optional*) - sinalizador ```True/False```, indicando se a pressão da onda de choque deve ser calculada usando a fórmula, ou apenas usar a pressão estática ```'p_0a '`` ` (*valor padrão ```Verdadeiro```*).
    - ```'p_0a'```(*opcional*) - pressão do ar na frente do projétil, Pa (*valor padrão 100.000*).
    - ```'k_air'```(*opcional*) - índice adiabático do ar (*valor padrão 1,4*).
    - ```'c_0a'```(*opcional*) - velocidade do som no ar, m/s (*valor padrão 340*).
- ```'heat'``` - **seção opcional**. Esta seção armazena um dicionário com dados iniciais relativos à troca de calor da bomba hidráulica com o barril. Se você não especificar este elemento, todos os valores padrão serão usados. O dicionário possui os seguintes elementos:
    - ```'enabled'```(*optional*) - flag ```True/False```, indicando se a troca de calor com o barril deve ser levada em consideração (*valor padrão ```True``` *).
    - ```'heat_barrel'```(*optional*) - flag ```True/False```, indicando se é necessário levar em conta o cálculo dinâmico da temperatura da parede do barril, ou se o a temperatura das paredes do barril não muda (*valor padrão `` `True```*).
    - ```'F_0'```(*opcional*) - área inicial de transferência de calor, m^2 (*valor padrão 4W_0/d*).
    - ```'Pr'```(*opcional*) - Número Prandl (*valor padrão 0,74*).
    - ```'T_w0'```(*opcional*) - temperatura inicial da parede, K. Se não for especificada, então a temperatura inicial será medida.
    - ```'mu_0'```(*opcional*) - coeficiente de viscosidade dinâmica dos gases em pó para a fórmula de Sutherland, Pa*s (*valor padrão 0,175e-4*).
    - ```'T_cs'```(*opcional*) - também para a fórmula de Sutherland, K (*valor padrão 628*).
    - ```'T_0s'```(*opcional*) - também para a fórmula de Sutherland, K (*valor padrão 273*).
    - ```'c_b'```(*opcional*) - capacidade térmica do material do barril, J/(kg * deg) (*valor padrão 500*).
    - ```'rho_b'```(*opcional*) - densidade do material do barril, kg/m^3 (*valor padrão 7900*).
    - ```'lambda_b'```(*opcional*) - condutividade térmica do material do cano, W/(m graus) (*valor padrão 40*).
    - ```'lambda_g'```(*opcional*) - condutividade térmica de gases em pó, W/(m K) (*valor padrão 0,2218*).
- ```'stop_conditions'``` - **seção obrigatória**. Esta seção armazena um dicionário com dados iniciais relacionados às condições para o final do cálculo. **Deve ter pelo menos um elemento** (qualquer um dos seguintes). Se diversas condições forem especificadas, o cálculo será interrompido devido à condição que foi acionada primeiro. O dicionário possui os seguintes elementos:
    - ```'t_max'```(*opcional*) - s, interrompe o cálculo quando t > t_max.
    - ```'steps_max'```(*opcional*) - faça no máximo etapas de integração steps_max.
    - ```'v_p'```(*opcional*) - m/s, interrompe o cálculo quando a velocidade do projétil atingir v_p.
    - ```'x_p'```(*opcional*) - m, interrompe o cálculo quando o projétil tiver passado x_p metros (no momento inicial o projétil passou 0 m).
    - ```'p_max'```(*opcional*) - Pa, interrompa o cálculo se a pressão ultrapassar p_max.
- ```'meta_termo'``` - **seção obrigatória para cálculos termodinâmicos**. Esta seção armazena um dicionário com dados iniciais relacionados aos metaparâmetros do cálculo termodinâmico. O dicionário possui os seguintes elementos:
    - ```'dt'``` - s, intervalo de tempo.
    - ```'method'```(*opcional*) - método de integração. Opções possíveis: Euler - 'euler'; Runge-Kutty 2ª ordem -'rk2'; - Runge-Kutty 4ª ordem - 'rk4' (*valor padrão 'rk2'*).
- ```'meta_lagrange'``` - **seção obrigatória para cálculos gasodinâmicos**. Esta seção armazena um dicionário com dados iniciais relacionados aos metaparâmetros do cálculo gasodinâmico. O dicionário possui os seguintes elementos:
    - ```'n_cells'``` - número de células da grade.
    - ```'CFL'``` - Número do Courant (0 < CFL < 1).
    - ```'W'```(*opcional*) - requisito adicional para aumentar a estabilidade: o próximo passo de tempo não pode ser W vezes maior que o atual.

## Estrutura dos resultados do modelo termodinâmico

Dependendo dos resultados do cálculo, o dicionário pode ser de dois tipos. Caso ocorra algum erro no cálculo, será gerado o seguinte dicionário:

```python
{
     'stop_reason': 'error',    # indica que ocorreu um erro durante o processo de cálculo
     'error_message': '...',    # descrição do erro
     'exception': Error('...'), # referência ao próprio erro (pode ser levantado usando raise para rastrear)
     'execution_time': float    # tempo de execução da função em segundos
}
```

Exemplo:

```python 
result = ozvb_termo({})  #passa um dicionário vazio
print(result)
>>> {
    'stop_reason': 'error',
    'error_message': 'O dicionário opts deve ter um campo "powders", que especifica os parâmetros a carregar. Um exemplo de dicionário de opções correto pode ser obtido na função get_termo_options_sample()',
    'exception': ValueError('O dicionário de opções deve ter um campo "powders", que especifica os parâmetros de cobrança. Um exemplo de dicionário de opções correto pode ser obtido na função get_termo_options_sample()'),
    'execution_time': 1.7400000047018693e-05
}
```

Se o cálculo ocorreu sem erros, o dicionário com os resultados serão o seguinte:

```python
{
     't': np.array([...]),      # array numpy com pontos de tempo em segundos em que os valores restantes foram calculados
     'p_m': np.array([...]),    # array numpy com pressão balística média em Pa
     'T': np.array([...]),      # array numpy com temperatura GPS em Kelvin
     'x_p':np.array([...]),     # array numpy com posição do projétil em metros (no momento inicial x_p==0)
     'v_p': np.array([...]),    # array numpy com velocidade do projétil em m/s (no momento inicial v_p==0)
     'Q_pa': np.array([...]),   # numpy array com a energia total em J gasta na superação das forças de resistência à pressão atmosférica na frente do projétil
     'Q_w': np.array([...]),    # array numpy com a energia total em J dada pelo GPS para aquecer o cano
     'W_p': np.array([...]),    # numpy array com volume do projétil em m^3
     'W_c': np.array([...]),    # numpy array com volume em m^3 ocupado pelo covolume GPS e pela fase condensada GPS
     'T_w': np.array([...]),    # matriz numpy com temperatura média do cano em K
     'k': np.array([...]),      # matriz numpy com expoentes adiabáticos GPS
     'z_1': np.array([...]),    # array numpy com a espessura relativa do cofre queimado da amostra de pólvora nº 1
     'psi_1': np.array([...]),  # numpy array com a massa relativa da pólvora queimado da amostra nº 1
     'z_2': np.array([...]),    # numpy array com a espessura relativa do cofre queimado da amostra de pólvora nº 2
     'psi_2': np.array([...]),  # array numpy com a massa relativa do pó queimado da amostra nº 2
         ...                    # e assim por diante N vezes
     'stop_reason': str,        # motivo para parar o cálculo ('t_max', 'steps_max', 'v_p', 'x_p', 'p_max')
     'execution_time': float    # tempo gasto no cálculo, em segundos
}
```
Exemplo:
```python

opts = get_options_sample()
result = ozvb_termo(opts)
print(result)
>>> {
    't':    array([0.   , 0.   , ..., 0.027, 0.027]),
    'p_m':  array([ 1000000.   ,  1002189.433, ..., 90680294.893, 90629603.46 ]),
    'T':    array([2427.   , 2427.487, ..., 1824.249, 1823.988]),
    'x_p':  array([0.   , 0.   , ..., 6.394, 6.398]),
    'v_p':  array([  0.   ,   0.   , ..., 689.994, 690.085]),
    'Q_pa': array([    0.   ,     0.   , ..., 45159.509, 45195.554]),
    'Q_w':  array([      0.   ,       0.   , ..., 3447622.549, 3449318.738]),
    'W_p':  array([0.033, 0.033, ..., 0.107, 0.107]),
    'W_c':  array([0.008, 0.008, ..., 0.014, 0.014]),
    'T_w':  array([293.15 , 293.15 , ..., 315.661, 315.661]),
    'k':    array([1.22 , 1.22 , ..., 1.238, 1.238]),
    'z_1':  array([0.   , 0.   , ..., 0.954, 0.954]),
    'psi_1':array([0.   , 0.   , ..., 0.954, 0.954]),
    'z_2':  array([0.   , 0.   , ..., 1.343, 1.343]),
    'psi_2':array([0.   , 0.   , ..., 0.987, 0.987]),
    'stop_reason': 'v_p',
    'execution_time': 0.21484209999971426
}
```

## Estrutura dos resultados do modelo dinâmico de gases


Dependendo dos resultados do cálculo, o dicionário pode ser de dois tipos. Caso ocorra algum erro no cálculo, será gerado o seguinte dicionário:

```python
{
    'stop_reason': 'error',     # indica que ocorreu um erro durante o processo de cálculo
     'error_message': '...',    # descrição do erro
     'exception': Error('...'), # referência ao próprio erro (pode ser levantado usando raise para rastrear)
     'execution_time': float    # tempo de execução da função em segundos
}
```

Exemplo:

```python
result = ozvb_lagrange({})  # passa um dicionário vazio
print(result)
>>> {
    'stop_reason': 'erro',
    'error_message': 'O dicionário opts deve conter o campo "powders", que especifica os parâmetros de cobrança. Um exemplo de dicionário de opções correto pode ser obtido na função get_termo_options_sample()',
    'exception': ValueError('O dicionário opts deve conter o campo "powders", que especifica os parâmetros de cobrança. Um exemplo de dicionário opts correto pode ser obtido na função get_termo_options_sample()'),
    'execution_time': 1.7400000047018693e-05
}
```


Se o cálculo ocorreu sem erros, o dicionário com os resultados será o seguinte:

```python
{
    'stop_reason': str,         # motivo para parar o cálculo ('t_max', 'steps_max', 'v_p', 'x_p', 'p_max')
    'execution_time': float,    # tempo de execução do cálculo em segundos
    'layers': [                 # lista com dicionários. Cada dicionário armazena dados de uma camada de tempo
        {                       # Dicionário da primeira camada temporária. A camada consiste em N células
            't': 0,0,           # tempo da camada de tempo em segundos
            'step_count': 0,    # número do passo de tempo
            'x': np.array([...]), # array numpy de coordenadas ao longo do comprimento dos nós da grade em metros, comprimento do array N+1
            'u': np.array([...]), # array numpy com velocidades do nó da grade em m/s, comprimento do array N+1
            'T': np.array([...]), # array numpy com temperaturas GPS em células em Kelvin. Comprimento da matriz N
            'rho': np.array([...]), # array numpy com densidades GPS em células em kg/m^3. Comprimento da matriz N
            'p': np.array([...]),   # array numpy com pressões GPS em células em Pa. Comprimento da matriz N
            'T_w':np.array([...]),  # matriz numpy com temperaturas da parede do cano em células em Kelvin. Comprimento da matriz N
            'k': np.array([...]),   # array numpy com expoentes adiabáticos GPS nas células. Comprimento da matriz N
            'z_1': np.array([...]), # numpy array com as espessuras relativas do cofre queimado da amostra de pólvora nº 1 por célula. Comprimento da matriz N
            'psi_1': np.array([...]),# numpy array com as massas relativas da pólvora queimado da amostra nº 1 por células. Comprimento da matriz N
            'z_2':np.array([...]),  # numpy array com as espessuras relativas do cofre queimado da amostra de pólvora nº 2 por célula. Comprimento da matriz N
            'psi_2': np.array([...]),# numpy array com as massas relativas da pólvora queimado da amostra nº 2 por células. Comprimento da matriz N
            ... # e assim por diante para todas as amostras
        },
        {...}, # Dicionário da segunda camada de tempo. A camada consiste em N células
        {...}, # Dicionário da terceira camada de tempo. A camada consiste em N células
        ...,                   e etc.
    ]     # fim da lista de 'layers'
}
```

Exemplo:

```python
opts = get_options_sample()
result = ozvb_lagrange(opts)
print(result)
>>> {
    'stop_reason': 'v_p',
    'execution_time': 0.167843300000186, 
    'layers': [
        {
            't': 0.0,
            'step_count': 0,
            'x': array([-2.78 , -2.762, ..., -0.019,  0.   ]),
            'u': array([0., 0., ..., 0., 0.]),
            'T': array([2427., 2427., ..., 2427., 2427.]),
            'rho': array([402.851, 402.851, ..., 402.851, 402.851]),
            'p': array([1000000., 1000000., ..., 1000000., 1000000.]),
            'T_w': array([293.15, 293.15, ..., 293.15, 293.15]),
            'k': array([1.22, 1.22, ..., 1.22, 1.22]),
            'z_1': array([0., 0., ..., 0., 0.]),
            'psi_1': array([0., 0., ..., 0., 0.]),
            'z_2': array([0., 0., ..., 0., 0.]),
            'psi_2': array([0., 0., ..., 0., 0.])
        },
        {
            't': 0.00026096741712768897,
            'step_count': 1,
            'x': array([-2.78 , -2.762, ..., -0.019,  0.   ]),
            'u': array([0., 0., ..., 0., 0.]),
            'T': array([2450.216, 2450.216, ..., 2450.216, 2450.216]),
            'rho': array([402.851, 402.851, ..., 402.851, 402.851]),
            'p': array([1114231.986, 1114231.986, ..., 1114231.986, 1114231.986]),
            'T_w': array([293.15, 293.15, ..., 293.15, 293.15]),
            'k': array([1.222, 1.222, ..., 1.222, 1.222]),
            'z_1': array([0., 0., ..., 0., 0.]),
            'psi_1': array([0., 0., ..., 0., 0.]),
            'z_2': array([0., 0., ..., 0., 0.]),
            'psi_2': array([0., 0., ..., 0., 0.])
        },
        ...
    ]
}
```

## Funções adicionais

A biblioteca também possui diversas funções adicionais, cuja descrição está em sua [documentação](https://devman.org/qna/13/chto-takoe-docstring-s-chem-ego-edjat/):

```python
from pyballistics import get_full_options, get_db_powder, get_powder_names

print(get_full_options.__doc__)
print(get_db_powder.__doc__)
print(get_powder_names.__doc__)
```
