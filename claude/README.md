# WeSpeak

Connect your hotel's WeSpeak workspace to ChatGPT or Claude. The plugin combines the WeSpeak customer MCP with a skill that explains how to use its tools, interpret their results, and carry out common hotel workflows.

## English

### What you can do

- Find guest conversations and read their message history.
- Review leads, funnel stages, commercial metrics, and recorded pixel conversions.
- Look up reservations recorded in WeSpeak Payments when that module is enabled.
- Review and update assistant knowledge, behavior, promotions, and hotel services when your role allows it.
- Manage conversations and lead stages, investigate assistant issues, and submit a report to WeSpeak operations when requested.
- Find instructions in the WeSpeak help center.

The available tools depend on your WeSpeak role and enabled modules. This is an operator tool for hotel teams with an existing WeSpeak account. It does not provide a general hotel booking service or access to another customer's workspace.

### Connect your account

1. Install the plugin in your AI application's plugin directory, or import the preview package provided by WeSpeak. A preview package does not imply a public listing has been approved.
2. Open the plugin's connection settings and connect **WeSpeak**. Sign in through the WeSpeak authorization page with the account that has access to your hotel.
3. Your WeSpeak account needs MCP access enabled. Without it the sign-in itself completes, but WeSpeak stays disconnected or keeps asking you to sign in again: contact WeSpeak support if that happens.
4. Start a new conversation and select WeSpeak. First verify which workspace the connection identifies, then request a read-only task such as finding a help article.

The package supplies the public OAuth client ID; it contains no account password, API key, or access token. If your host asks for an OAuth client ID, use the connection details in the **WeSpeak MCP Connection** card, under **Settings** in WeSpeak. Leave the client secret blank and choose public-client authentication (`none`) where prompted.

For Claude Code, open `/mcp` after installing the plugin and authenticate the WeSpeak server. For Claude/Cowork, connect it in the plugin's **Connectors** tab. Installing the package does not complete account authorization.

For multiple WeSpeak accounts, use each account's connector URL from its own **WeSpeak MCP Connection** card and keep the connections separately named. A generic directory connection uses the organization in the authorized session; changing a username in a prompt does not change that authorization.

### Try it

- “Summarize this week's guest conversations and show the coverage of your analysis.”
- “Find open leads without activity for three days and suggest follow-up priorities.”
- “Review the assistant's current settings and suggest improvements without making changes.”

Ask explicitly when you want to change data, contact operations, or run an assistant simulation. Simulation and diagnostic tools can exercise configured email, booking, or payment flows; they are not a substitute for a read-only review. The plugin explains the result of each requested action and respects the tools available to your role.

## Español

Conecta la cuenta de tu hotel en WeSpeak para revisar conversaciones, analizar resultados, dar seguimiento a oportunidades y gestionar el conocimiento y la configuración del asistente según tus permisos.

Instala el plugin, conecta WeSpeak e inicia sesión con la cuenta de tu hotel. Necesitas tener habilitado el acceso MCP. En Claude/Cowork, completa la conexión en la pestaña **Connectors**; en Claude Code, utiliza `/mcp`. Si gestionas varias cuentas, utiliza las direcciones de conexión de la tarjeta **WeSpeak MCP Connection**, en **Configuración** dentro de WeSpeak, y verifica la cuenta antes de operar.

Ejemplos:

- “Resume las conversaciones de esta semana e indica qué parte pudiste revisar”.
- “Busca oportunidades abiertas sin actividad durante tres días y sugiere prioridades”.
- “Revisa la configuración del asistente y propone mejoras sin aplicar cambios”.

Solicita expresamente los cambios, los reportes a operaciones y las simulaciones del asistente. Las herramientas de simulación y diagnóstico pueden ejecutar flujos configurados de correo, reservas o pagos; no equivalen a una consulta de solo lectura.

## Português (Brasil)

Conecte a conta do seu hotel no WeSpeak para revisar conversas, analisar resultados, acompanhar oportunidades e gerenciar o conhecimento e as configurações do assistente conforme suas permissões.

Instale o plugin, conecte o WeSpeak e entre com a conta do seu hotel. O acesso MCP precisa estar habilitado. No Claude/Cowork, conclua a conexão na aba **Connectors**; no Claude Code, use `/mcp`. Se você gerencia várias contas, use os endereços de conexão do cartão **WeSpeak MCP Connection**, em **Configurações** no WeSpeak, e confira a conta antes de agir.

Exemplos:

- “Resuma as conversas desta semana e indique o alcance da análise”.
- “Encontre oportunidades abertas sem atividade há três dias e sugira prioridades”.
- “Revise as configurações do assistente e sugira melhorias sem aplicar alterações”.

Solicite explicitamente alterações, relatos para a equipe de operações e simulações do assistente. As ferramentas de simulação e diagnóstico podem executar fluxos configurados de e-mail, reservas ou pagamentos; não são consultas somente de leitura.

## Support and service policies

- [WeSpeak](https://www.wespeak.pro/)
- [Support](https://www.wespeak.pro/docs/soporte/)
- [Privacy policy](https://www.wespeak.pro/avisodeprivacidad/)
- [Terms of service](https://www.wespeak.pro/terminosycondiciones/)

## Plugin source license

The plugin's skills and configuration files are available under the MIT License, copyright NextGen Conversations Corp (WeSpeak). [LICENSE](LICENSE) identifies the covered files and contains the full terms. Both distribution packages include this license.

Logos, icons, other images, and trademarks are excluded from the MIT grant. The license does not cover the WeSpeak platform, remote MCP implementation, hosted services, or other repository files. Access to WeSpeak still requires an authorized account and remains subject to the service policies above.

A preview package does not indicate directory approval or publication.
