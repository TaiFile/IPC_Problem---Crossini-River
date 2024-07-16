# Problema da Travessia do Rio

Este problema faz parte de um conjunto de problemas escrito por Anthony Joseph da U.C. Berkeley, mas não sei se ele é o autor original. É semelhante ao problema H2O no sentido de que é um tipo peculiar de barreira que só permite que threads passem em certas combinações.

Em algum lugar perto de Redmond, Washington, há um barco a remo que é usado tanto por hackers de Linux quanto por funcionários (servos) da Microsoft para atravessar um rio. O barco pode levar exatamente quatro pessoas; ele não sai da margem com mais ou menos que isso. Para garantir a segurança dos passageiros, não é permitido colocar um hacker no barco com três servos, nem colocar um servo com três hackers. Qualquer outra combinação é segura.

À medida que cada thread embarca no barco, ela deve invocar uma função chamada `embarcar`. Você deve garantir que todas as quatro threads de cada carga de barco invoquem `embarcar` antes que qualquer uma das threads da próxima carga de barco o façam.

Depois que todas as quatro threads invocarem `embarcar`, exatamente uma delas deve chamar uma função chamada `remarBarco`, indicando que essa thread irá remar. Não importa qual thread chama a função, desde que uma o faça.

Não se preocupe com a direção da viagem. Suponha que estamos interessados apenas no tráfego indo em uma direção.

## Dica para a Travessia do Rio

Aqui estão as variáveis que usei na minha solução:

```python
barreira = Barrier(4)
mutex = Semaphore(1)
hackers = 0
servos = 0
filaHackers = Semaphore(0)
filaServos = Semaphore(0)
local éCapitão = False
```

hackers e servos contam o número de hackers e servos esperando para embarcar. Como ambos são protegidos por mutex, podemos verificar a condição de ambas as variáveis sem nos preocupar com uma atualização inoportuna. Este é outro exemplo de um placar.

filaHackers e filaServos nos permitem controlar o número de hackers e servos que passam. A barreira garante que todas as quatro threads tenham invocado embarcar antes que o capitão invoque remarBarco.

éCapitão é uma variável local que indica qual thread deve invocar remar.

##Solução para Travessia do Rio
A ideia básica desta solução é que cada chegada atualiza um dos contadores e depois verifica se completa um grupo, seja por ser o quarto de seu tipo ou completando um par misto.

Apresentarei o código para hackers; o código dos servos é simétrico (exceto, claro, que é 1000 vezes maior, cheio de bugs e contém um navegador web embutido)
```python
mutex.wait()
hackers += 1
if hackers == 4:
    filaHackers.signal(4)
    hackers = 0
    éCapitão = True
elif hackers == 2 and servos >= 2:
    filaHackers.signal(2)
    filaServos.signal(2)
    servos -= 2
    hackers = 0
    éCapitão = True
else:
    mutex.signal() # capitão mantém o mutex

filaHackers.wait()

embarcar()
barreira.wait()

if éCapitão:
    remarBarco()
    mutex.signal() # capitão libera o mutex
```

À medida que cada thread passa pela seção de exclusão mútua, verifica se uma tripulação completa está pronta para embarcar. Se sim, sinaliza as threads apropriadas, declara-se capitão e mantém o mutex para barrar threads adicionais até que o barco tenha partido.

A barreira mantém o controle de quantas threads embarcaram. Quando a última thread chega, todas as threads procedem. O capitão invoca remar e então (finalmente) libera o mutex.
