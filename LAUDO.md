# LAUDO TÉCNICO — Operação Resgate

## Bug 1 — main.dart (import de firebase_options.dart)
- **Bug:** 1
- **Arquivo:** lib/main.dart
- **Sintoma:** O app não compila/abre: `Target of URI doesn't exist: firebase_options.dart`.
- **Causa:** O arquivo gerado pelo FlutterFire CLI não está no repositório (README avisa), então `DefaultFirebaseOptions` não existe.
- **Correção:** Gerado `lib/firebase_options.dart` com `flutterfire configure` no projeto Firebase. Também substituído `test/widget_test.dart` (referenciava `MyApp` e pacote inexistentes e quebrava o analisador).
- **Teste:** `flutter pub get`, `flutter analyze` sem erros e `flutter run -d chrome` abre a tela de login.

## Bug 2 — DetalhePage sem argumento obrigatório
- **Bug:** 2
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Erro de compilação ao tocar no item: argumento `ocorrencia` obrigatório não informado.
- **Causa:** `DetalhePage()` foi chamado sem passar a ocorrência.
- **Correção:** `DetalhePage(ocorrencia: o)`.
- **Teste:** Toquei em uma ocorrência e a tela de detalhes abriu com os dados corretos.

## Bug 3 — autorId lido do campo errado
- **Bug:** 3
- **Arquivo:** lib/models/ocorrencia.dart
- **Sintoma:** Botão de alterar status nunca aparecia / dono incorreto.
- **Causa:** `autorId` era lido de `d['autorNome']`.
- **Correção:** `autorId: d['autorId']`.
- **Teste:** Na conta dona o botão 'Marcar como resolvido' aparece; na outra conta não.

## Bug 4 — Status invertido na leitura
- **Bug:** 4
- **Arquivo:** lib/models/ocorrencia.dart
- **Sintoma:** Casos abertos apareciam como resolvidos e vice-versa.
- **Causa:** `resolvido: !(d['resolvido'] ...)` negava o valor.
- **Correção:** `resolvido: d['resolvido'] as bool? ?? false`.
- **Teste:** Registrei um item (aparece 'Em aberto') e marquei como resolvido (aparece 'Resolvido').

## Bug 5 — Dono salvo com e-mail em vez de uid
- **Bug:** 5
- **Arquivo:** lib/services/central_service.dart (registrar)
- **Sintoma:** O dono nunca era reconhecido; comparação com `u.uid` sempre falhava.
- **Causa:** `autorId` gravava `u.email`, mas a verificação usa `u.uid`.
- **Correção:** `'autorId': u.uid`.
- **Teste:** No Firestore o campo `autorId` do documento novo é igual ao uid do usuário.

## Bug 6 — Alterar status não inverte o valor
- **Bug:** 6
- **Arquivo:** lib/services/central_service.dart (alterarStatus)
- **Sintoma:** Ao clicar, o status gravado não mudava.
- **Causa:** Gravava `item.resolvido` (valor atual) em vez do oposto.
- **Correção:** `update({'resolvido': !item.resolvido})`.
- **Teste:** Clicar em resolver/reabrir alterna o status e o campo muda no Firestore.

## Bug 7 — Pista salva na ocorrência errada
- **Bug:** 7
- **Arquivo:** lib/services/central_service.dart (enviarPista)
- **Sintoma:** Pista enviada não aparecia na ocorrência; todas iam para um mesmo lugar.
- **Causa:** Usava `.doc('pistas')` fixo em vez do id da ocorrência.
- **Correção:** `.doc(ocorrenciaId).collection('pistas')`.
- **Teste:** Na conta B enviei uma pista; ela apareceu na ocorrência correta nas contas A e B.

## Bug 8 — Envio de pista quebrado (condição invertida e texto vazio)
- **Bug:** 8
- **Arquivo:** lib/pages/detalhe_page.dart (enviar)
- **Sintoma:** Com texto digitado nada era enviado; sem texto a função continuava e enviava pista vazia.
- **Causa:** `isNotEmpty` retornava cedo e a mensagem enviada era `''` fixa.
- **Correção:** `if (pista.text.trim().isEmpty) return;` e envio de `pista.text`.
- **Teste:** Digitei uma pista, enviei e ela apareceu na lista com o texto; campo vazio não envia.

## Bug 9 — Switch do filtro invertido
- **Bug:** 9
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Ligar 'Somente não resolvidos' desligava o filtro.
- **Causa:** `somenteAbertos = !value`.
- **Correção:** `somenteAbertos = value`.
- **Teste:** O switch ligado mantém o estado ligado e o filtro é aplicado.

## Bug 10 — Filtro mostra os resolvidos
- **Bug:** 10
- **Arquivo:** lib/pages/central_page.dart
- **Sintoma:** Com o filtro ativo apareciam só os resolvidos.
- **Causa:** `where((o) => o.resolvido)` em vez de `!o.resolvido`.
- **Correção:** `where((o) => !o.resolvido)`.
- **Teste:** Com um item resolvido e outro aberto, o filtro mostra só o aberto.

## Bug 11 — Validação do formulário com && 
- **Bug:** 11
- **Arquivo:** lib/pages/central_page.dart (cadastrar)
- **Sintoma:** Era possível cadastrar com título ou local vazio.
- **Causa:** Usava `&&`: só barrava se os dois estivessem vazios.
- **Correção:** Trocado por `||`.
- **Teste:** Deixei só o título vazio: aparece 'Informe título e local.' e nada é salvo.

## Bug 12 — Ordem das ocorrências
- **Bug:** 12
- **Arquivo:** lib/services/central_service.dart (acompanhar)
- **Sintoma:** Itens mais antigos apareciam no topo.
- **Causa:** `orderBy('criadoEm', descending: false)`.
- **Correção:** `descending: true`.
- **Teste:** Registrei dois itens e o mais recente aparece primeiro.

## Segurança
A autenticação foi mantida e foi adicionado o arquivo `firestore.rules` (leitura só para logados; alteração de status só pelo dono; pistas só criadas pelo próprio autor).
