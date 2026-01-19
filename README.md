# SvelteCraft

Uma reconstrução do Minecraft na web utilizando Svelte 5, Threlte e Three.js.

<center>

![SvelteCraft](src/lib/assets/svelte_craft.png)

</center>

## Sobre o Projeto

Este projeto foi desenvolvido como um experimento para avaliar as capacidades do modelo **Claude Opus 4.5** em tarefas de desenvolvimento de jogos e aplicações 3D interativas.

### Objetivos do Teste

O SvelteCraft foi utilizado para testar o Claude Opus nos seguintes quesitos:

| Quesito | Descrição |
|---------|-----------|
| **Geração Procedural** | Capacidade de gerar um mundo 3D proceduralmente utilizando Simplex Noise |
| **Diagnóstico de Performance** | Identificar e corrigir problemas de performance com poucas informações técnicas sobre o erro |
| **Custo-Benefício** | Avaliar a relação entre qualidade do resultado e tokens utilizados |
| **Integração de Assets** | Adicionar e configurar texturas corretamente no pipeline do Three.js |
| **Frame Rate vs Delta Time** | Validar compreensão da diferença entre movimento baseado em frame rate fixo e delta time para física consistente |
| **Frameworks Alternativos** | Testar a eficácia do modelo em frameworks menos convencionais como Svelte, considerando que o Sonnet 4.5 possui melhor desempenho em projetos Next.js |

## Tecnologias

- **Svelte 5** - Framework reativo com runes
- **SvelteKit** - Meta-framework para Svelte
- **Threlte** - Integração declarativa do Three.js para Svelte
- **Three.js** - Biblioteca 3D para WebGL
- **Tailwind CSS** - Estilização utilitária
- **TypeScript** - Tipagem estática

## Funcionalidades

- Geração procedural de terreno com Simplex Noise
- Sistema de blocos com múltiplas texturas (grama, terra, pedra, bedrock, etc.)
- Movimento em primeira pessoa com física básica (gravidade, pulo, colisão)
- Colocação e remoção de blocos
- Sistema de chunks para renderização otimizada
- Nuvens procedurais
- Ciclo dia/noite com sol dinâmico

## Executando o Projeto

```bash
# Instalar dependências
npm install

# Rodar em modo de desenvolvimento
npm run dev

# Build para produção
npm run build
```

## Controles

| Tecla | Ação |
|-------|------|
| W/A/S/D | Movimentação |
| Mouse | Rotação da câmera |
| Espaço | Pular |
| Shift | Correr |
| Clique Esquerdo | Remover bloco |
| Clique Direito | Colocar bloco |
| 1-6 | Selecionar bloco na hotbar |

## Aprendizados

Durante o desenvolvimento com Claude Opus, foram observados os seguintes pontos:

1. **Geração Procedural**: O modelo demonstrou compreensão sólida de algoritmos de noise e sua aplicação em terrenos 3D.

2. **Debug de Performance**: Ao reportar que "o jogo acelera em PCs mais potentes", o modelo identificou corretamente a ausência de delta time no game loop sem necessidade de detalhes técnicos adicionais.

3. **Frameworks Alternativos**: Mesmo com menor volume de dados de treinamento em Svelte comparado a React/Next.js, o modelo conseguiu trabalhar efetivamente com as particularidades do framework, incluindo a nova sintaxe de runes do Svelte 5.

## Licença

MIT
