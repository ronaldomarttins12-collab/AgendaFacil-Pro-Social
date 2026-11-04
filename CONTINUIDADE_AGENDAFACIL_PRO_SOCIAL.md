# AgendaFácil Pro Social — Documento de continuidade

**Data de referência:** 01/11/2026  
**Status:** beta integrada; não publicada em hospedagem e ainda não liberada para produção.

Este documento deve ser enviado junto com o ZIP do aplicativo quando o projeto for retomado em outro aparelho ou conversa.

## 1. Projeto e ambiente

- Projeto Supabase: `AgendaFacil-Pro-Social-SP`
- Project ref: `rmxhglhqcclzajpypzrk`
- Região: `sa-east-1` (São Paulo)
- Status observado: `ACTIVE_HEALTHY`
- Meta de orçamento definida pelo usuário: até **R$500**.
- Arquivo de aplicativo incluído neste pacote: `AgendaFacil_Pro_Social_BETA_REVISAO_COMPLETA_01-11-2026.zip`.

## 2. Regras essenciais do produto

1. Empresas de **serviços** oferecem agendamento; seus serviços não entram no carrinho.
2. Empresas de **produtos** oferecem catálogo, carrinho e pedidos.
3. Uma empresa pode ativar mais de uma modalidade: Delivery, Agendamentos e Loja Virtual, conforme o tipo de operação.
4. Busca prioriza produtos/serviços realmente cadastrados por empresas da cidade do consumidor. Empresas de outras cidades aparecem como alternativa quando não houver oferta local.
5. Taxa da plataforma para entrega local: R$0. O comerciante pode configurar sua própria taxa local.
6. Frete entre cidades é negociado diretamente entre vendedor e comprador; o comprador precisa aceitar a proposta antes do pedido avançar.
7. O app inclui área social, feed, Stories, curtidas, comentários, seguidores, perfis comerciais e mensagens.
8. A notificação discreta “Empresas perto de mim” deve aparecer 5 segundos após abrir a área do consumidor, a cada abertura, apenas com localização autorizada; depende de coordenadas cadastradas pelas empresas.
9. Gestão comercial inclui estoque, entradas/saídas, PDV, caixa, turnos, colaboradores, permissões e trilha de auditoria.

## 3. O que já está implementado

### Consumidor
- Fluxo online conectado ao Supabase para catálogo, empresas, posts, Stories, pedidos e agendamentos.
- Cadastro/edição de cidade e estado do perfil.
- Busca por itens reais do catálogo, com prioridade para a cidade do consumidor.
- Produtos só entram no carrinho quando a loja virtual está habilitada e há estoque.
- Serviços são apresentados no fluxo de agendamento, não no carrinho.
- Carrinho limitado a uma empresa por vez.
- Criação de pedido via RPC, reserva de estoque e acompanhamento de status.
- Taxa local do comerciante e frete intermunicipal com proposta/aceite.
- Criação e cancelamento de solicitações de agendamento; validação de horário comercial, duração e conflito de horários.
- Seguir empresas, curtir/comentar posts, visualizar Stories e iniciar conversas.
- Link compartilhável da empresa com parâmetro `?loja=slug`.
- Modo demonstração local separado do fluxo real.

### Comerciante e equipe
- Cadastro/edição/ativação de produtos e serviços.
- Configuração de horário semanal de funcionamento.
- Gestão de pedidos e agendamentos.
- Proposta de frete intermunicipal e atualização segura do status dos pedidos.
- PDV com múltiplos itens, desconto e forma de pagamento.
- Abertura e fechamento de caixa por operador, com saldo inicial, esperado, contado e diferença.
- Movimentações de caixa e estoque com operador identificado.
- Colaboradores vinculados a usuários já cadastrados, com papéis e bloqueio/reativação.
- Auditoria de operações.
- Solicitação de plano de gestão; ativação ainda manual, sem cobrança automática.

## 4. Atualização realizada nesta retomada

- Foi criada no Supabase a função `public.create_business_product(...)`, com validação de usuário e papel (owner/admin/manager), validação de preço/custo/quantidades e criação transacional do produto com estoque inicial.
- Quando há estoque inicial, a função registra também a movimentação de entrada por meio da rotina segura de estoque.
- Permissão de execução concedida a `authenticated` e negada a `anon`.
- Teste transacional de cadastro e estoque inicial executado com `ROLLBACK`; não deixa dados de teste persistidos.
- A migração correspondente está dentro do ZIP em `AgendaFacil_Pro_Social_Online/supabase/migrations/20261101195000_create_business_product_rpc.sql`.
- O projeto Supabase foi consultado novamente e permaneceu `ACTIVE_HEALTHY`.
- Verificação de integridade do ZIP: sem erros.

## 5. Testes já executados anteriormente

- Inserção de produto autenticado.
- Criação de pedido e itens, reserva e restauração de estoque ao cancelar.
- Fluxo de frete intermunicipal: proposta, aceite e atualização do total.
- Entrega local com taxa do comerciante e taxa da plataforma zerada.
- Criação de agendamento dentro do horário configurado.
- PDV, movimentação de caixa e fechamento com valor contado.
- Abertura de caixa após fechamento.
- Criação de empresa e criação automática de vínculo do proprietário/modalidades.
- Solicitação de plano e início de conversa/mensagem.
- `node --check` do JavaScript e `unzip -t` do pacote.

**Limitação de testes:** o teste completo em navegador Android/Chrome não foi concluído; a tentativa de Chromium headless expirou. Portanto, ainda é necessária validação prática com contas separadas de consumidor, proprietário e colaborador.

## 6. Pendências para uma etapa posterior

### Ações manuais do usuário / serviços externos
- Publicar o ZIP em hospedagem HTTPS e configurar domínio.
- Configurar SMTP e testar confirmação/recuperação de senha.
- Ativar proteção contra senhas vazadas no Supabase Auth, se disponível no plano contratado.
- Escolher provedor de pagamentos e fornecer/autorizar credenciais; ainda não há checkout com cobrança automática nem cobrança automática de planos.
- Criar bucket de Storage e políticas RLS para upload de fotos/vídeos; atualmente imagens dependem de URLs.

### Desenvolvimento e validação ainda necessários
- Convites automáticos de colaboradores por e-mail (atualmente vinculação manual por UUID de usuário cadastrado).
- Disponibilidade individual de funcionários, feriados, folgas e intervalos entre atendimentos.
- Testes reais em Android/Chrome com contas separadas e aparelhos diferentes.
- Termos, privacidade, exportação/exclusão de dados, moderação/denúncia, backups e monitoramento.
- Notificações push/tempo real.
- Geolocalização/reverse geocoding para preencher coordenadas comerciais de forma assistida.
- Revisão final de segurança das funções `SECURITY DEFINER` e das políticas antes de produção.

## 7. Alertas conhecidos do Supabase

- As tabelas do schema público foram verificadas com RLS habilitado.
- O Security Advisor sinalizou 15 funções `SECURITY DEFINER` executáveis por usuários autenticados. Elas são usadas para operações privilegiadas e contêm verificações internas de identidade/permissão; isso deve continuar sendo revisado e testado. Nenhuma função `SECURITY DEFINER` do schema público estava executável por `anon` na consulta feita.
- O Advisor também sinalizou proteção contra senhas vazadas desativada.
- O aviso de RLS sem políticas em `private.platform_admins` refere-se a tabela do schema privado e foi classificado como informativo.
- O Performance Advisor mostrou avisos de índices ainda não utilizados; em base pequena isso pode ocorrer por falta de volume de consultas e não é motivo isolado para remover índices.

## 8. Como retomar

1. Enviar este documento e o ZIP do aplicativo na nova conversa.
2. Dizer: **“Vamos continuar o AgendaFácil Pro Social a partir deste documento. Preserve as regras do produto e trabalhe primeiro nas pendências que não exigem minha ação manual.”**
3. Não usar protótipos/ZIPs antigos. A base de continuidade é o ZIP beta revisado incluído neste pacote.

## 9. Arquivos incluídos

- `AgendaFacil_Pro_Social_BETA_REVISAO_COMPLETA_01-11-2026.zip` — aplicativo beta e migração SQL mais recente.
- `CONTINUIDADE_AGENDAFACIL_PRO_SOCIAL_01-11-2026.md` — este resumo de decisões, estado, testes e pendências.
