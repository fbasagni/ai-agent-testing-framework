
def rodar_framework_testes():
    print("==================================================")
    print("      INICIANDO FRAMEWORK DE AI EVALUATION        ")
    print("==================================================\n")
    
    with open('dataset_testes.json', 'r', encoding='utf-8') as f:
        casos_teste = json.load(f)

    bugs_encontrados = []

    for caso in casos_teste:
        print(f"▶️ [Caso #{caso['id']}] - {caso['pilar']}")
        resposta_luna = agente_ecommerce_roupas([], caso['input_usuario'])
        resultado_auditoria = llm_as_a_judge(caso['input_usuario'], resposta_luna, caso['criterio_aceite'])
        
        # Exibe no terminal as novas métricas estruturadas
        status = "✅ PASSOU" if resultado_auditoria.get("passou") else "❌ FALHOU"
        print(f"📊 Veredicto: {status} | Score Final: {resultado_auditoria.get('score_final')}/5")
        print(f"🛡️  Métricas: {resultado_auditoria.get('metricas')}")
        print(f"📝 Justificativa: {resultado_auditoria.get('justificativa')}")
        print("-" * 60 + "\n")

        # Se o teste falhar, nós capturamos o BUG
        if not resultado_auditoria.get("passou"):
            bugs_encontrados.append({
                "id": caso['id'],
                "pilar": caso['pilar'],
                "cenario": caso['cenario'],
                "input_usuario": caso['input_usuario'],
                "resposta_luna": resposta_luna,
                "justificativa_juiz": resultado_auditoria.get('justificativa'),
                "metricas": resultado_auditoria.get('metricas')
            })

    # GERAÇÃO AUTOMÁTICA DO RELATÓRIO DE BUGS (Se houver falhas)
    if bugs_encontrados:
        print(f"⚠️ {len(bugs_encontrados)} falha(s) detectada(s)! Gerando relatório de bugs...")
        with open("relatorio_bugs_ia.md", "w", encoding="utf-8") as f:
            f.write("# 🐛 Relatório de Bugs de IA - Falha em Produção\n\n")
            f.write(f"Este relatório foi gerado automaticamente pelo framework de QA.\n\n")
            
            for bug in bugs_encontrados:
                f.write(f"## Bug no Pilar: {bug['pilar']}\n")
                f.write(f"- **Cenário:** {bug['cenario']}\n")
                f.write(f"- **Input de Ataque do Usuário:** `{bug['input_usuario']}`\n")
                f.write(f"- **Resposta com Falha da Luna:** *\"{bug['resposta_luna'].strip()}\"*\n")
                f.write(f"- **Análise do Juiz (Motivo do Bug):** {bug['justificativa_juiz']}\n")
                f.write(f"- **Métricas do Erro:** {bug['metricas']}\n")
                f.write("\n" + "="*40 + "\n\n")
        print("📁 Arquivo 'relatorio_bugs_ia.md' criado com sucesso com as evidências!")
    else:
        print("🎉 Excelente! Nenhum bug foi detectado nos cenários de teste.")