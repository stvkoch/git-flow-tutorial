

#Decentralized but centralized


![git flow](http://nvie.com/img/2010/01/centr-decentr.png)


The repository setup that we use and that works well with this branching model, is that with a central “truth” repo. Note that this repo is only considered to be the central one (since Git is a DVCS, there is no such thing as a central repo at a technical level). We will refer to this repo as origin, since this name is familiar to all Git users.



Each developer pulls and pushes to origin. But besides the centralized push-pull relationships, each developer may also pull changes from other peers to form sub teams. For example, this might be useful to work together with two or more developers on a big new feature, before pushing the work in progress to origin prematurely. In the figure above, there are subteams of Alice and Bob, Alice and David, and Clair and David.

Technically, this means nothing more than that Alice has defined a Git remote, named bob, pointing to Bob’s repository, and vice versa.



##O 'branch' principal


Basicamente o modelo de desenvolvimento é inspirado nos modelos que existem por ai. Ter dois _branch_ principais por todo o ciclo de vida do software.

 * master
 * develop

 ![git flow](http://nvie.com/img/2009/12/bm002.png)

O _branch_ _master_ e _origin_ devem ser familiar a todos os utilizadores _Git_, outro branch existente é chamado de _develop_.


Nós consideramos o _origin/master_ o _branch_ principal onde o codigo fonte marcado como _HEAD_ **sempre** reflete o código que está em produção.

Consideramos que no _branch_ _origin/develop_ o _commit_ _HEAD_ sempre tem o codigo fonte com as mudanças necessarias para o próximo lançamento (_release_). Alguns chamam este _branch_ de "integration branch". Isto é, onde qualquer importante modificações (nightly builds) estão armazenadas.


Quando o código fonte no _branch_ _develop_ atinge o estado de estar pronto para ser lançado "release", todas as mudanças devem ser mescladas (_merge_) com o _branch_ _master_ e tags com o numero da versão.


Portanto, cada ve que um _merge_ ocorre dentro do _master_, ou seja uma nova definição de uma versão. Nós tendemos ser muitos rigorosos quando a isso, e teoricamente poderiamos usar _hooks_ do _Git_ para disparar automaticamente _scripts_ toda vez que é feito um commit para o _branch_ _master_.




##Supporting branches

Assim como _branches_ principais _master e _develop_, nosso modelo de desenvolvimento usa uma variadade de _branches_ para suportar desenvolvimento paralelos entre os membros de nosso time de desenvolvedores, e facilmente rastrear _features_, preparas _merges_ e corrigir rapidamente erros no codigo em produção. Diferente dos _branches_ principais, estes _branches_ *sempre tem um ciclo de vida limitado*, e serão eventualmente removidos.


Os diferentes tipos de _branches_ que podemos usar são:

 * _Feature branches_
 * _Release branches_
 * _Hotfix branches_


Cada um destes _branches_ tem um propósito específico e são obrigado a seguir regras rigorosas quanto a qual _branch_ deve ser usar origem e a qual _branch_ podem se mesclar (_merge_).



##_Feature branches_


![git flow](http://nvie.com/img/2009/12/fb.png)

Rules:
 * Só pode ter como origem o _branch_: develop
 * Pode somente fazer _merge_ com: _develop_
 * Pode dar o nome que quiser excepto  _master_, _develop_, _release-*_, ou _hotfix-*_


_Feature branches_ (as vezes chamado  _topic branches_) são usado para desenvolver novas _features_ ou distantes lançamentos de novas versões. 
Ao iniciar o desenvolvimento de um recurso, a versão de destino em que este recurso será incorporado pode ser bem desconhecido no momento momento que começa o desenvolvimento. A essencia do _Feature branch_ é que ele existe durante o desenvolvimento do recurso, e eventualmente será mesclado _merge_ de volta para o _branch_ _develop_ (para depois ser adicionado ao _branch_ _master_) ou descartado.

*_Feature branches_* tipicamente existe somente no repositório local e não é enviado para o repositório _origin_ (remote ...)






###Criando um _feature branch_


Quando começamos a trabalhar em um novo recurso o _branch_ deve vir do _branch_ _develop_.


	$ git checkout -b myfeature develop
	#Switched to a new branch "myfeature"



###Incorporando _feature_ no _develop branch_


Terminado o desenvolvimento do recurso é hora de mescla-lo (_merge_) de volta para o _branch_ de _develop_ e finalmente adicionar a versão que será lançada:


	$ git checkout develop
	#Switched to branch 'develop'
	$ git merge --no-ff myfeature
	#Updating ea1b82a..05e9557
	#(Summary of changes)
	$ git branch -d myfeature
	#Deleted branch myfeature (was 05e9557).
	$ git push origin develop



A _flag_ --no-ff diz para o _merge_ sempre criar um novo _commit_, mesmo que o _merge_ podesse ser criado com o _fast-forward_. Isso evita a perda de informações sobre a existência histórica de um ramo de recurso e agrupa todas as submissões que juntos foi adicionado o recurso. Compare:


![git flow](http://nvie.com/img/2010/01/merge-without-ff.png)


No ultimo caso é impossivel dizeres olhando para o _Git history_ qual o commit (e _branch_) que implementou o novo recurso. Voce teria que ler manualmente todas as mensagens de log. Reverter um grupo de _commits_ é uma chatisse que poderia evitar usando --no-ff.


Sim, ele vai criar mais alguns objetos _commits_ vazios, mas o ganho é muito maior que a que custo.  Infelizmente, não encontrei uma maneira de fazer --no-ff o comportamento padrão de git merge ainda, mas deve ser realmente.





##Release branches


Rules:
 * Só pode ter como origem o _branch_: develop
 * Pode somente fazer _merge_ com: _develop_ e _master_
 * O nome do _branch_ deve começar com _release-*_



_Release branches_ suportam o codigo que será preparado lançamente de uma nova versão a produção. Estes _branches_ permitem que sejam planejados os 'I's e os 'T's. Além disso, eles permitem correções de correção bugs e preparação de meta-dados para uma versão (número de versão, datas de construção, etc.). Fazendo todo esse trabalho em um ramo de lançamento, o ramo de desenvolvimento é desmarcado para receber recursos para o próximo grande lançamento.




O momento chave para criar um novo _release branch_ é quando o _branch develop_ reflete o estado desejado da nova versão. Pelo menos todas os recursos planeados estão no _branch_ _develop_. Todas os recursos que estão planeados para futuras versões devem esperar até depois que o _release branch_.


É exatamente no inicio do _release branch_ que é atribuido um numero de versão, e não anteriormente. Até este momento o _branch_ _develop_ reflete todas as mudanças para a próxima versão, mas não está claro se a próxima versão eventualmente ser tornará 0.3 ou 1.0 que o _branch_ _release_ seja iniciado.




###Criando _release branch_



_Release branches_ são criado *a partir do* _branch develop_. Por exemplo, digamos que a versão 1.1.5 é a versão lançada a produção e há planeado um grande lançamento. O estado do desenvolvimento está pronto para o próximo lançamento e decidimos que a versão será a 1.2 (ao inves de 1.1.6 ou 2.0).Então ramificamos o _branch_ _develop_ para um novo _release_ _branch_ com um nome que reflete a nova versão:


	$ git checkout -b release-1.2 develop
	#Switched to a new branch "release-1.2"
	$ ./bump-version.sh 1.2
	#Files modified successfully, version bumped to 1.2.
	$ git commit -a -m "Bumped version number to 1.2"
	#[release-1.2 74d9424] Bumped version number to 1.2
	#1 files changed, 1 insertions(+), 1 deletions(-)


Depois de de criar o novo _branch_ e mudarmos para ele. Aqui o _bump-version.sh_ é um ficticio _shell script_ que muda alguns ficheiros para refletir a nova versão. (Voce consegue fazer isso manualmente) Então confirmamos a nova versão fazendo um _commit_.


Este novo _branch_ pode existir durante um tempo consideravel até que o lançamente tenha sido completado definitivamente. Durante este tempo pode ocorrer correcções de _bugs_ que podem ser aplicadas a diretamente a este _branch_ (e não sobre o _branch_ _develop_). Adicionar novas funcionalidades são estritamente proibidas e devem aguardar o proximo grande lançamento.




###Terminando _release branch_


Quando achares que o _release_ _branch_ actual está pronto para ser tornar em um lançamento real, algumas acções precisam ser tomadas. Primeiro, o _release branch_ é mesclado (_merge_) ao _master branch_ (desde que cada _commit_ do _branch_ _master seja um lançamento de uma nova versão). Em seguida, o _commit_ deve ser marcado com uma _tag_ para referencias hitoricas. E finalmente as alterações feitas no _release_ _branch_ precisam ser mescladas de volta para o _branch_ _develop_, para que os futuros lançamentos tenhas também as alterações e correcções deste lançamento.


As primeiros duas etapas do _Git_:


	$ git checkout master
	#Altera para branch 'master'
	$ git merge --no-ff release-1.2
	#Mescla as mudanças do branch release-1.2 para o branch atual recursivamente.
	#(Summary of changes)
	$ git tag -a 1.2



O lançamento está agora concluido e marcado para futuras referencias.
Talvez queira usar as _flags_ -s ou -u para assinar criptograficamente a _tag_.




Para mantes as mudanças feitas no _release_ _branch_, nós precisamos mescla-las ao _branch_ _develop_:

	$ git checkout develop
	#Altera para o branch 'develop'
	$ git merge --no-ff release-1.2
	#Mescla as mudanças do branch release-1.2 para o branch atual recursivamente.
	#(Summary of changes)


Esta etapa pode levar a alguns conflitos, se assim for corrija-os e faço o commit.

Agora estamos realmente prontos e o _release_ _branch_ pode ser removido. Nós não precisamos mais dele.


	$ git branch -d release-1.2
	#Deleted branch release-1.2 (was ff452fe).






##Hotfix branches


![git flow](http://nvie.com/img/2010/01/hotfix-branches1.png)


