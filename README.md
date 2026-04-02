## 🛠️ O Arsenal Tecnológico: Por que usamos cada ferramenta?

No desenvolvimento de mods para jogos modernos (especialmente os que usam **IL2CPP**), trabalhamos em uma camada entre o código gerenciado (C#) e o código nativo (C++). Abaixo está a função de cada "app" no nosso fluxo de trabalho:

### 1. BepInEx (Unity IL2CPP Build)
* **O que é:** O "Coração" do Modding. É um framework de código aberto que atua como um carregador de plugins.
* **Por que usamos:** Jogos comerciais são "fechados". O BepInEx faz o *Bootstrap* (inicialização forçada), injetando-se dentro do processo do jogo antes mesmo dele carregar a primeira tela. Ele cria a pasta `Plugins`, onde nossa DLL reside, e gerencia o console de depuração que usamos para ver os logs em tempo real.

### 2. HarmonyLib (HarmonyX)
* **O que é:** Uma biblioteca de *Patching* de métodos em tempo de execução.
* **Por que usamos:** Em ADS, aprendemos que não devemos alterar o código-fonte original se pudermos estendê-lo. O Harmony nos permite fazer isso sem ter o código-fonte. Ele usa uma técnica chamada **Hooking**: ele "sequestra" a chamada de uma função do jogo (ex: `TakeDamage`), executa o nosso código primeiro e depois decide se a função original deve ou não continuar.

### 3. Il2CppInterop (Unhollower/MelonLoader base)
* **O que é:** Um tradutor de tipos entre C++ e C#.
* **Por que usamos:** Como o jogo foi convertido para C++ (IL2CPP), o C# "puro" não consegue entender onde estão as classes. O Interop gera as **Dummy DLLs** (DLLs de referência). Elas não contêm o código do jogo, mas contêm a "assinatura" (nomes de classes e métodos) para que o Visual Studio não dê erro de compilação ao digitarmos `MyPlayer`.

### 4. Unity PhysicsModule (Assembly Modular)
* **O que é:** Um módulo específico do motor Unity que cuida de colisões e gravidade.
* **Por que usamos:** Em versões antigas do Unity, tudo ficava em um arquivo só. Hoje, a física é separada. Identificamos que para o nosso mod de "No-Clip" ou "Ghost Mode" funcionar, precisávamos entender como o `PhysicsModule` interage com o `PlayerController`, garantindo que o jogador não caia através do mapa ao desativar colisões com inimigos.

### 5. dnSpy / ILSpy
* **O que é:** Um descompilador e depurador de assemblies .NET.
* **Por que usamos:** É a nossa "Lanterna". Usamos o dnSpy para ler as Dummy DLLs geradas e descobrir os nomes exatos dos métodos que queremos patchear. Sem ele, estaríamos tentando adivinhar nomes de funções no escuro.

---

## 🔄 Fluxo de Execução do Mod (Pipeline)

1.  **Injeção:** O `BepInEx` detecta nossa DLL na pasta `/plugins`.
2.  **Mapeamento:** O `Il2CppInterop` faz a ponte para o mod entender os objetos do jogo na memória RAM.
3.  **Redirecionamento:** O `Harmony` aplica os patches nos métodos de Vida e Movimentação.
4.  **Interação:** O jogador aperta `F9`, o `Update` detecta a entrada e altera as variáveis de estado no motor Unity.
