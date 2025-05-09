# Manual Avançado de RxJS

### Visão Geral do RxJS

RxJS (Reactive Extensions for JavaScript) é uma biblioteca para composição de programas assíncronos e baseados em eventos usando sequências observáveis (observables). Seu ponto forte é a manipulação declarativa de streams de dados.

## 1. Transformação de Dados

### Operadores: `map`, `pluck`, `tap`, `scan`

### Conceito:

Permitem transformar, interceptar ou acumular dados emitidos por observables.

### **Operadores de Transformação de Dados**

### `map`

**O que faz:** Transforma o valor emitido, como o `.map()` do JavaScript.

**Quando usar:**

Quando você quer modificar o valor antes de usar ou exibir.

**Exemplo prático:**

Você recebe um usuário da API e quer exibir o nome em letras maiúsculas:

```tsx
this.user$.pipe(
  map(user => user.name.toUpperCase())
)
```

### `pluc`

**O que faz:** Extrai diretamente uma ou mais propriedades de um objeto emitido.

**Quando usar:**

Quando você só precisa de uma propriedade específica de um objeto.

**Exemplo prático:**

Você quer pegar só o `email` de um objeto `user` que vem da API:

```tsx

this.user$.pipe(
  pluck('email')
)
```

Também pode pegar propriedades aninhadas:

```tsx
pluck('endereco', 'cidade') // user.endereco.cidade
```

### `tap` (antes chamado `do`)

**O que faz:** Executa uma ação com o valor, **sem alterar o valor**.

**Quando usar:**

Para **logar**, **debugar** ou **acionar efeitos colaterais** sem mexer no fluxo.

**Exemplo prático:**

Você quer ver o valor antes de fazer algo com ele:

```tsx
this.user$.pipe(
  tap(user => console.log('Usuário recebido:', user))
)
```

### `scan`

**O que faz:** Funciona como um `reduce` reativo, acumulando valores emitidos ao longo do tempo.

**Quando usar:**

Quando você quer **acumular** dados ou **manter estado** entre emissões.

**Exemplo prático:**

Contador de cliques:

```tsx
this.click$.pipe(
  scan((acc, _) => acc + 1, 0)
)
```

Outro exemplo útil: construir uma lista incrementando dados:

```tsx
this.items$.pipe(
  scan((list, item) => [...list, item], [])
)
```

---

## 2. Controle de Tempo

### Operadores: `debounceTime`, `throttleTime`, `delay`, `auditTime`

### Conceito:

Manipulam o tempo entre as emissões, ideais para eventos de UI ou streams de alta frequência.

### Operadores de Controle de Tempo

### `debounceTime`

**O que faz:** Espera um intervalo de tempo **sem novas emissões** antes de emitir o valor final.

**Quando usar:**

Quando o usuário **digita** e você quer evitar chamadas para cada letra (ex: busca em tempo real).

**Exemplo prático:**

Só envia a busca se o usuário parar de digitar por 300ms:

```tsx
this.searchInput$.pipe(
  debounceTime(300)
)
```

 **Ideal para:** Inputs de busca, autocomplete, filtros em tempo real. 

### `throttleTime`

**O que faz:** Emite o primeiro valor e **ignora os próximos** durante o tempo definido.

**Quando usar:**

Quando você quer **limitar a frequência** de execuções — mesmo que o usuário dispare muitos eventos.

**Exemplo prático:**

Evitar múltiplos cliques num botão de envio:

```tsx
this.buttonClick$.pipe(
  throttleTime(1000)
)
```

 **Ideal para:** Botões, scroll, resize, movmentações rápidas. 

### `delay`

**O que faz:** Apenas **atrasa** a emissão de um valor por um tempo fixo.

**Quando usar:**

Quando você quer simular atraso, como carregamento fake ou esperar por alguma animação.

**Exemplo prático:**

Aguardar 1 segundo antes de exibir uma mensagem de sucesso:

```tsx
this.success$.pipe(
  delay(1000)
)
```

### `auditTime`

**O que faz:** Espera um tempo fixo e **emite o último valor** que chegou nesse período.

**Quando usar:**

Quando você quer capturar **só o valor final** de uma sequência rápida, com controle de janela.

**Exemplo prático:**

Monitorar o scroll e emitir a **posição final** a cada 1 segundo:

```tsx
this.scroll$.pipe(
  auditTime(1000)
)
```

 **Diferença entre `auditTime` e `throttleTime`:** 

- `throttleTime`: emite o **primeiro** e ignora o resto por um tempo.
- `auditTime`: ignora tudo e emite o **último** após o tempo.

---

## 3. Encadeamento de Observables

### Operadores: `switchMap`, `mergeMap`, `concatMap`, `exhaustMap`

### Conceito:

Permitem combinar ou substituir observables emitidos dinamicamente (ex: ao clicar ou digitar).

### **Operadores de Encadeamento de Observables**

Esses operadores **recebem um valor** e disparam **um novo observable** com base nele. A diferença está em **como lidam com múltiplas emissões** simultâneas.

### `switchMap`

**O que faz:** Cancela a requisição anterior e usa **apenas a mais recente**.

**Quando usar:**

Quando o valor anterior **não é mais relevante** — ex: autocomplete, login.

**Exemplo prático:**

Usuário digita em um campo de busca:

```tsx
this.searchInput$.pipe(
  debounceTime(300),
  switchMap(term => this.api.search(term))
)
```

 **Ideal para:** Busca em tempo real, login, requisições dependentes do input mais recente. 

### `mergeMap`

**O que faz:** Executa **todos os observables ao mesmo tempo**, em paralelo.

**Quando usar:**

Quando você **quer lidar com todas as requisições**, sem cancelar nenhuma.

**Exemplo prático:**

Usuário envia múltiplos arquivos para upload:

```tsx
from(files).pipe(
  mergeMap(file => this.api.upload(file))
)
```

 **Ideal para:** Uploads múltiplos, notificações em tempo real, processamento paralelo. 

### `concatMap`

**O que faz:** Executa **um por vez, em ordem**. Espera o anterior terminar.

**Quando usar:**

Quando a **ordem importa** ou quando você **precisa evitar sobrecarga**.

**Exemplo prático:**

Registrar logs sequenciais ou enviar dados em ordem:

```tsx
from(events).pipe(
  concatMap(event => this.api.logEvent(event))
)
```

 **Ideal para:** Logs, requisições que dependem da ordem, APIs que rejeitam chamadas paralelas. 

### `exhaustMap`

**O que faz:** Ignora novas emissões **enquanto uma requisição estiver em andamento**.

**Quando usar:**

Quando você **não quer múltiplas execuções simultâneas**, como evitar spam de cliques.

**Exemplo prático:**

Botão de envio que não deve ser clicado múltiplas vezes:

```tsx
this.submit$.pipe(
  exhaustMap(() => this.api.submitForm())
)
```

 **Ideal para:** Submissão de formulários, ações que não devem ser repetidas até concluir. 

---

### 4. Tratamento de Erros e Repetição

### Operadores: `retry`, `retryWhen`, `catchError`

### `catchError`

**O que faz:** Captura erros de um observable e **trata ou substitui por outro fluxo**.

**Quando usar:**

Sempre que você quer evitar que um erro **quebre o fluxo** inteiro.

**Exemplo prático:**

Mostrar mensagem de erro e continuar com observable vazio:

```tsx
this.api.getUser().pipe(
  catchError(error => {
    this.toast.show('Erro ao carregar usuário');
    return of(null); // ou EMPTY
  })
)
```

 **Ideal para:** Mostrar fallback, logs de erro, evitar quebra da UI. 

### `retry`

**O que faz:** Em caso de erro, **refaz automaticamente** a operação N vezes.

**Quando usar:**

Quando erros são passageiros, como falhas de rede momentâneas.

**Exemplo prático:**

Tentar carregar o usuário 3 vezes antes de falhar:

```tsx
this.api.getUser().pipe(
  retry(3)
)
```

**💡 Ideal para:** Requisições instáveis, chamadas que têm chance de funcionar ao tentar de novo.

### `retryWhen`

**O que faz:** Permite **controlar quando e como repetir**, com lógica personalizada.

**Quando usar:**

Quando você precisa de **retries com delay**, **exponencial backoff**, ou lógica condicional.

**Exemplo prático:**

Tentar novamente a cada 2 segundos, até 3 vezes:

```tsx
this.api.getUser().pipe(
  retryWhen(errors =>
    errors.pipe(
      delay(2000),
      take(3)
    )
  )
)
```

  **Ideal para:** APIs instáveis com backoff, serviços que precisam de espera entre tentativas. 

## 5. Filtragem

### Operadores: `filter`, `take`, `first`, `skip`, `distinctUntilChanged`

### `filter`

**O que faz:** Só deixa passar valores que **satisfazem uma condição**.

**Quando usar:**

Quando você quer ignorar certas emissões que **não interessam**.

**Exemplo prático:**

Deixar passar apenas usuários com acesso de admin:

```tsx
this.users$.pipe(
  filter(user => user.role === 'admin')
)
```

 **Ideal para:** Restringir por condição, ignorar valores indesejados. 

### `take`

**O que faz:** Deixa passar **apenas os N primeiros valores**, e depois completa.

**Quando usar:**

Quando você só quer escutar **uma ou poucas vezes**.

**Exemplo prático:**

Pegar **apenas o primeiro valor** de um evento:

```tsx
this.buttonClick$.pipe(
  take(1)
)
```

 **Ideal para:** Unsubscribe automático, eventos únicos. 

### `first`

**O que faz:** Emite **o primeiro valor** (com ou sem condição), depois completa.

**Quando usar:**

Quando você quer **só o primeiro resultado útil**.

**Exemplo prático:**

Pegar o primeiro item que tem estoque:

```tsx
this.items$.pipe(
  first(item => item.inStock)
)
```

 **Ideal para:** Primeira ocorrência que satisfaz uma condição. 

### `skip`

**O que faz:** **Ignora os N primeiros valores**, e depois começa a emitir.

**Quando usar:**

Quando os primeiros valores são irrelevantes (ex: placeholders).

**Exemplo prático:**

Ignorar o primeiro clique (teste, setup):

```tsx
this.clicks$.pipe(
  skip(1)
)
```

 **Ideal para:** Ignorar inicializações, valores prévios. 

### `distinctUntilChanged`

**O que faz:** Só emite **se o valor for diferente do anterior**.

**Quando usar:**

Quando você quer evitar **repetições consecutivas**.

**Exemplo prático:**

Evitar disparo de chamada se valor do input não mudou:

```tsx
this.searchInput$.pipe(
  distinctUntilChanged()
)
```

 **Ideal para:** Inputs, formulários, estados que mudam pouco. 

## 6. Combinação de Observables

### Operadores: `combineLatest`, `withLatestFrom`, `forkJoin`, `zip`

Esses operadores servem para **juntar valores de múltiplas fontes**. A diferença entre eles está **quando** e **como** eles combinam esses valores.

`combineLatest`

**O que faz:** Emite um array (ou objeto) com os **últimos valores de cada observable**, sempre que **qualquer um** deles emite.

**Quando usar:**

Quando você precisa da combinação dos **valores mais recentes de várias fontes**.

**Exemplo prático:**

Atualizar a UI com filtros combinados:

```tsx
combineLatest([
  this.category$,
  this.searchTerm$
]).subscribe(([category, term]) => {
  this.filterProducts(category, term);
});
```

 **Ideal para:** Filtros dinâmicos, formulários dependentes de múltiplas fontes. 

### `withLatestFrom`

**O que faz:** Quando o observable principal emite, **pega os últimos valores dos outros**, **mas só emite quando o principal emitir**.

**Quando usar:**

Quando **só um observable importa como gatilho**, e você precisa do contexto dos outros.

**Exemplo prático:**

Ao clicar num botão, enviar o formulário com os dados mais recentes:

```tsx
this.submit$.pipe(
  withLatestFrom(this.formData$)
).subscribe(([_, formData]) => {
  this.api.submit(formData);
});
```

 **Ideal para:** Botões que usam dados atuais, eventos que precisam de estado no momento da ação. 

### `forkJoin`

**O que faz:** Executa vários observables em paralelo e **emite todos os resultados de uma vez**, **somente quando todos completarem**.

**Quando usar:**

Quando você quer fazer **requisições paralelas** e só prosseguir **quando todas terminarem com sucesso**.

**Exemplo prático:**

Buscar dados de perfil e configurações ao mesmo tempo:

```tsx
forkJoin({
  profile: this.api.getProfile(),
  settings: this.api.getSettings()
}).subscribe(result => {
  this.profile = result.profile;
  this.settings = result.settings;
});
```

 **Ideal para:** Requisições independentes que precisam ser feitas juntas (ex: dashboard). 

### `zip`

**O que faz:** Combina os observables **um a um**, como um zíper — só emite quando **cada um emitiu um valor**.

**Quando usar:**

Quando você precisa **sincronizar valores por posição**.

**Exemplo prático:**

Animar pares de dados na mesma ordem:

```tsx
zip(this.names$, this.ages$).subscribe(([name, age]) => {
  console.log(`${name} tem ${age} anos`);
});
```

 **Ideal para:** Fluxos que devem andar juntos, par a par (ordem importa). 

## 7. Limpeza e Cancelamento

### Operadores: `takeUntil`, `unsubscribe`, `finalize`

### `takeUntil`

**O que faz:** Fica ouvindo até que **outro observable dispare**, e então **cancela automaticamente**.

**Quando usar:**

Quando você quer **desligar** um fluxo com base em outro evento (como `ngOnDestroy`).

**Exemplo prático:**

Cancelar observables ao destruir um componente:

```tsx
ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}

this.api.getData().pipe(
  takeUntil(this.destroy$)
).subscribe();
```

 **Ideal para:** Cancelar streams em componentes Angular, timers, websockets, etc. 

### `unsubscribe`

**O que faz:** **Cancela manualmente** a inscrição (subscription) de um observable.

**Quando usar:**

Quando você precisa de controle direto ou está fora de um pipe.

**Exemplo prático:**

```tsx
const sub = this.api.getData().subscribe(data => console.log(data));
// depois...
sub.unsubscribe();
```

 **Ideal para:** Subscrições fora de templates ou onde o operador `takeUntil` não se aplica. 

### `finalize`

**O que faz:** Executa **uma ação final** quando o observable **completa ou é cancelado** (inclusive com erro ou `unsubscribe`).

**Quando usar:**

Quando você precisa **encerrar loaders, conexões, ou limpar estado**.

**Exemplo prático:**

Mostrar loading até finalizar o observable:

```
this.api.getData().pipe(
  finalize(() => this.loading = false)
).subscribe();
```

 **Ideal para:** Limpeza, logs, loaders — qualquer pós-processamento. 

## 8. Sujeitos (Subjects)

### Tipos:

- `Subject`: não guarda estado
- `BehaviorSubject`: guarda o último valor
- `ReplaySubject`: reenvia valores anteriores
- `AsyncSubject`: emite apenas o último quando completo

### `Subject`

**O que faz:** Um canal de emissão que **não guarda valor anterior**.

**Quando usar:**

Quando **estado anterior não importa**, só precisa notificar algo **agora**.

**Exemplo prático:**

Notificar clique de botão, sem se importar com histórico:

```tsx
const click$ = new Subject<void>();

this.button.nativeElement.addEventListener('click', () => {
  click$.next();
});
```

 **Ideal para:** Eventos pontuais, notificações. 

### `ReplaySubject`

**O que faz:** Reenvia **valores antigos** para novos inscritos. Pode definir **quantos valores** reter.

**Quando usar:**

Quando você quer que os novos inscritos **"replayem" o histórico** de dados.

**Exemplo prático:**

Histórico de mensagens:

```tsx
const messages$ = new ReplaySubject<string>(3); // reenvia últimas 3

messages$.next('msg1');
messages$.next('msg2');
messages$.next('msg3');

messages$.subscribe(msg => console.log('Recebido:', msg));
// Recebe msg1, msg2, msg3
```

 **Ideal para:** Chat, notificações, histórico. 

### `AsyncSubject`

**O que faz:** Só emite **o último valor** **quando o observable completa**.

**Quando usar:**

Quando você só quer entregar **o resultado final de um processo** (ex: uma requisição).

**Exemplo prático:**

Execução de um job que só importa o valor final:

```tsx

const job$ = new AsyncSubject<number>();

job$.subscribe(console.log); // só recebe valor no complete

job$.next(1);
job$.next(2);
job$.next(3);

job$.complete(); // emite: 3
```

 **Ideal para:** Processos que só têm valor útil no final (upload finalizado, cálculo terminado, etc.). 

## 9. Criação de Observables

### Funções: `of`, `from`, `interval`, `timer`, `defer`

### Quando usar:

- `of`: emitir valores estáticos
- `from`: converter promises/arrays
- `interval`: emissões periódicas
- `timer`: delay com ou sem repetição
- `defer`: gerar observable na hora da subscrição

### `of(...)`

**O que faz:** Cria um observable que **emite os valores fornecidos** (de forma síncrona) e **completa**.

**Quando usar:**

Para **emitir valores simples**, estáticos ou de teste.

**Exemplo prático:**

```tsx
of(1, 2, 3).subscribe(console.log); // Emite: 1, 2, 3
```

 **Ideal para:** Dados de mock, testes, valores imediatos. 

### `from(...)`

**O que faz:** Converte **array, promise, iterable ou observable-like** em observable.

**Quando usar:**

Para transformar dados externos em observables reativos.

**Exemplo prático:**

```tsx
from(fetch('/api/user').then(res => res.json()))
  .subscribe(console.log)
```

 Ideal para: Promises (requisições), arrays, integrações externas. 

### `interval(ms)`

**O que faz:** Emite valores **sequenciais (0, 1, 2...) em intervalos regulares** de tempo.

**Quando usar:**

Para criar **pulsos temporais constantes**.

**Exemplo prático:**

```tsx
interval(1000).subscribe(val => console.log(`Segundos: ${val}`));
```

 **Ideal para:** Timers, animações, polling. 

### `timer(delay, intervalo?)`

**O que faz:** Emite um valor após um **delay**, ou de forma periódica **após o primeiro delay**.

**Quando usar:**

Quando você quer **esperar antes de começar**, com ou sem repetição.

**Exemplo prático:**

```tsx
// emite uma vez após 2 segundos
timer(2000).subscribe(() => console.log('2 segundos passaram'));

// emite a cada segundo, começando após 3s
timer(3000, 1000).subscribe(console.log);
```

 **Ideal para:** Delays controlados, timeout + loops. 

### `defer(() => observable)`

**O que faz:** Cria o observable **só na hora da inscrição**, garantindo que seja **executado dinamicamente**.

**Quando usar:**

Quando o observable precisa ser **gerado fresco** para cada inscrição (ex: dados que mudam a cada chamada).

**Exemplo prático:**

```tsx
const random$ = defer(() => of(Math.random()));

random$.subscribe(console.log); // número aleatório diferente a cada inscrição
```

 **Ideal para:** Observables dinâmicos, com lógica no momento da subscrição.
