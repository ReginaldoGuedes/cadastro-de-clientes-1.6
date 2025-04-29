  # cadastro-de-clientes-1.6
  Cadastro de Clientes Simples com Lista, pesquisável por id, nome, cpf e cnpj
  Desenvilvido em Python e PyQt5

  Segue o código: Os ícones com 24x24px em png pode buscar no site flatcons ou icons8, ícone do label formato .ico fica a seu critério adicionar, Imagem Logo com 200x100px em png.

    import sys
    import os
    import shutil
    import re
    import sqlite3
    import requests
    from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QWidget, QVBoxLayout, QHBoxLayout,
    QLabel, QLineEdit, QPushButton, QTableWidget, QTableWidgetItem,
    QMessageBox, QGridLayout, QProgressBar
    )
    from PyQt5.QtCore import Qt, QTimer, QPropertyAnimation, QRect
    from PyQt5.QtGui import QIcon
    from PyQt5.QtGui import QPixmap
    from PyQt5.QtCore import QSize # Importação movida para onde QSize é usado


    def resource_path(relative_path):
    """Obtém o caminho absoluto para o recurso, funcional para desenvolvimento e executável."""
    try:
        # PyInstaller cria uma pasta temporária e armazena o caminho em _MEIPASS
        base_path = sys._MEIPASS
    except Exception:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)


    class CadastroApp(QMainWindow):
      def __init__(self):
        super().__init__()
        self.setWindowTitle("Sistema de Cadastro | Gráfica Card Online")
        self.setGeometry(100, 100, 900, 700)
        
        # Define o ícone da janela
        self.setWindowIcon(QIcon(resource_path("icons/ico.ico")))

        # Aplicar estilo global
        self.setStyleSheet("""
        QMainWindow {
            background-color: #f0f4f8; /* Fundo claro */
        }
        QLabel {
            font-size: 12px;
            color: #333333; /* Texto escuro */
        }
        QLineEdit {
            background-color: #ffffff;
            border: 1px solid #cccccc;
            border-radius: 5px;
            padding: 5px;
        }
        QPushButton {
            background-color: #4CAF50; /* Verde */
            color: white;
            border: none;
            border-radius: 5px;
            padding: 8px 16px;
            font-size: 12px;
        }
        QPushButton:hover {
            background-color: #45a049; /* Verde mais escuro ao passar o mouse */
        }
        QTableWidget {
            background-color: #ffffff;
            border: 1px solid #cccccc;
            border-radius: 5px;
            alternate-background-color: #f9f9f9; /* Cor alternada para linhas */
        }
        QHeaderView::section {
            background-color: #4CAF50; /* Cabeçalho da tabela */
            color: white;
            padding: 5px;
            border: 1px solid #cccccc;
        }
        """)

        # Conexão com o banco de dados
        self.conn = sqlite3.connect("clientes_bd.bd")
        self.cursor = self.conn.cursor()
        self.criar_tabela()

        # Layout principal
        self.central_widget = QWidget()
        self.setCentralWidget(self.central_widget)
        self.layout = QVBoxLayout()
        self.central_widget.setLayout(self.layout)

        # Layout superior com logotipo
        logo_layout = QHBoxLayout()
        logo_label = QLabel()
        pixmap = QPixmap(resource_path("images/logo.png")) # Substitua "images/logo.png" pelo caminho da imagem
        logo_label.setPixmap(pixmap.scaled(200, 100, Qt.KeepAspectRatio))  # Redimensiona o logotipo
        logo_layout.addWidget(logo_label)
        logo_layout.setAlignment(Qt.AlignCenter)  # Centraliza o logotipo
        self.layout.addLayout(logo_layout)

        # Adicionar uma barra de progresso
        self.progress_bar = QProgressBar()
        self.progress_bar.setMaximum(100)
        self.progress_bar.setValue(0)
        self.progress_bar.setFixedHeight(7)  # Altura fixa da barra

        # Estilo personalizado para ajustar o tamanho do texto
        self.progress_bar.setStyleSheet("""
        QProgressBar {
            height: 7px; /* Altura da barra */
            background-color: #f0f4f8;
                border: 1px solid #cccccc;
                border-radius: 3px; /* Borda arredondada */
                text-align: right; /* Centraliza o texto */
                font-size: 7px; /* Tamanho reduzido da fonte */
            }
            QProgressBar::chunk {
                background-color: #4CAF50; /* Cor do progresso */
                    border-radius: 3px; /* Borda arredondada do progresso */
                }
                """)

        self.layout.addWidget(self.progress_bar)

        # Simular progresso
        for i in range(101):
            self.progress_bar.setValue(i)
            QApplication.processEvents()  # Atualiza a interface

            self.tabela = QTableWidget() # Inicialização movida para antes do uso
            self.tabela.setSortingEnabled(True)  # Habilita a ordenação por colunas

        # Adicionar espaçamento entre os elementos
        self.layout.setSpacing(10)  # Espaçamento entre os widgets
        self.botoes_layout = QHBoxLayout() # Inicialização movida para antes do uso
        self.botoes_layout.setSpacing(10)

        # Animar o tamanho de um widget
        self.botao_salvar = QPushButton("Salvar/Alterar") # Inicialização movida para antes do uso
        
        animation = QPropertyAnimation(self.botao_salvar, b"geometry")  # Replace 'widget' with 'self.botao_salvar' or the desired widget
        animation.setDuration(1000)  # Set duration in milliseconds
        animation.setStartValue(QRect(0, 0, 100, 30))  # Starting geometry
        animation.setEndValue(QRect(100, 100, 200, 60))  # Ending geometry
        animation.start()

        # Barra de botões (parte superior)
        # adição de ícone ao botão "Buscar"
        self.botao_buscar = QPushButton("Buscar") # Inicialização movida para antes do uso
        self.botao_buscar.setIcon(QIcon(resource_path("icons/search.png")))  # Substitua "icons/search.png" pelo caminho do ícone
        self.botao_buscar.setIconSize(QSize(24, 24))  # Define o tamanho do ícone

        # adição de ícone ao botão "Salvar"
        self.botao_salvar = QPushButton("Salvar/Alterar")
        self.botao_salvar.setIcon(QIcon(resource_path("icons/save.png")))
        self.botao_salvar.setIconSize(QSize(24, 24))
                
        self.botao_cadastrar_novo = QPushButton("Novo Cadastro")
        self.botao_cadastrar_novo.setIcon(QIcon(resource_path("icons/register.png")))
        self.botao_cadastrar_novo.setIconSize(QSize(24, 24))
        
        #self.botao_alterar = QPushButton("Alterar")
        #self.botao_alterar.setIcon(QIcon("icons/alterar.png"))
        #self.botao_alterar.setIconSize(QSize(24, 24))
        
        self.botao_apagar = QPushButton("Apagar")
        self.botao_apagar.setIcon(QIcon(resource_path("icons/trash.png")))
        self.botao_apagar.setIconSize(QSize(24, 24))
        
        self.botao_limpar = QPushButton("Limpar")
        self.botao_limpar.setIcon(QIcon(resource_path("icons/clean.png")))
        self.botao_limpar.setIconSize(QSize(24, 24))

        # Conectar os botões às suas funções
        self.botao_buscar.clicked.connect(self.buscar_e_preencher)
        self.botao_cadastrar_novo.clicked.connect(self.limpar_campos_para_cadastro)
        #self.botao_alterar.clicked.connect(self.alterar_dados)
        self.botao_apagar.clicked.connect(self.apagar_dados)
        self.botao_salvar.clicked.connect(self.salvar_dados)
        self.botao_limpar.clicked.connect(self.limpar_campos_e_deselecionar)

        # Adicionar botões ao layout
        self.botoes_layout.addWidget(self.botao_buscar)
        self.botoes_layout.addWidget(self.botao_cadastrar_novo)
        #self.botoes_layout.addWidget(self.botao_alterar)
        self.botoes_layout.addWidget(self.botao_apagar)
        self.botoes_layout.addWidget(self.botao_salvar)
        self.botoes_layout.addWidget(self.botao_limpar)
        self.layout.addLayout(self.botoes_layout)

        # Campo de pesquisa
        self.pesquisa_layout = QHBoxLayout()
        self.pesquisa_label = QLabel("Pesquisar:")
        self.campo_pesquisa = QLineEdit()
        self.campo_pesquisa.setPlaceholderText("Código, Nome, CPF/CNPJ")
        self.campo_pesquisa.returnPressed.connect(self.buscar_e_preencher)
        self.pesquisa_layout.addWidget(self.pesquisa_label)
        self.pesquisa_layout.addWidget(self.campo_pesquisa)
        self.layout.addLayout(self.pesquisa_layout)

        # Formulário de cadastro (grade 2x5)
        self.formulario_layout = QGridLayout()
        self.campos = {}
        campos_labels = [
            ("Nome/Razão", "nome"),
            ("CPF/CNPJ", "cpf_cnpj"),
            ("WhatsApp", "whatsapp"),
            ("Email", "email"),
            ("CEP", "cep"),
            ("Endereço", "endereco"),
            ("Nº", "numero"),
            ("Bairro", "bairro"),
            ("Cidade", "cidade")
        ]

        for idx, (label, campo) in enumerate(campos_labels):
            lbl = QLabel(label)
            txt = QLineEdit()
            row = idx // 3
            col = idx % 3
            self.formulario_layout.addWidget(lbl, row, col * 2)
            self.formulario_layout.addWidget(txt, row, col * 2 + 1)
            self.campos[campo] = txt

        # Conectar eventos de formatação e validação
        self.campos["cpf_cnpj"].textEdited.connect(self.formatar_cpf_cnpj)
        self.campos["whatsapp"].textEdited.connect(self.formatar_whatsapp)
        self.campos["cep"].textEdited.connect(self.formatar_cep)
        self.campos["email"].editingFinished.connect(self.validar_email)

        # Campo CEP com evento de busca automática
        self.campos["cep"].returnPressed.connect(self.preencher_endereco)

        # Adicionar formulário ao layout principal
        self.layout.addLayout(self.formulario_layout)

        # Tabela para exibir dados salvos
        # self.tabela = QTableWidget() # Já inicializado anteriormente
        self.tabela.setColumnCount(10)  # Uma coluna a mais para o ID
        self.tabela.setHorizontalHeaderLabels([
            "CÓD", "NOME/RAZÃO", "CPF/CNPJ", "WHATSAPP", "EMAIL", "ENDEREÇO", "Nº", "CEP", "BAIRRO", "CIDADE"
        ])
        self.tabela.itemClicked.connect(self.carregar_registro_selecionado)
        self.layout.addWidget(self.tabela)

        # Carregar dados na tabela
        self.carregar_dados()

        # Variável para rastrear o ID do registro sendo alterado
        self.registro_para_alterar = None
        
    def resource_path(relative_path):
        """Obtém o caminho absoluto para o recurso, funcional para desenvolvimento e executável."""
        try:
        # PyInstaller cria uma pasta temporária e armazena o caminho em _MEIPASS
            base_path = sys._MEIPASS
        except Exception:
            base_path = os.path.abspath(".")
            return os.path.join(base_path, relative_path)

    def formatar_cpf_cnpj(self):
        """Formata automaticamente o campo CPF/CNPJ."""
        campo = self.campos["cpf_cnpj"]
        texto = campo.text().replace(".", "").replace("-", "").replace("/", "")
        if len(texto) <= 11:  # CPF
            if len(texto) > 3:
                texto = f"{texto[:3]}.{texto[3:]}"
            if len(texto) > 7:
                texto = f"{texto[:7]}.{texto[7:]}"
            if len(texto) > 11:
                texto = f"{texto[:11]}-{texto[11:]}"
        else:  # CNPJ
            if len(texto) > 2:
                texto = f"{texto[:2]}.{texto[2:]}"
            if len(texto) > 6:
                texto = f"{texto[:6]}.{texto[6:]}"
            if len(texto) > 10:
                texto = f"{texto[:10]}/{texto[10:]}"
            if len(texto) > 15:
                texto = f"{texto[:15]}-{texto[15:]}"
        campo.setText(texto)
        campo.setCursorPosition(len(texto))  # Mantém o cursor no final

    def formatar_whatsapp(self):
        """Formata automaticamente o campo WhatsApp."""
        campo = self.campos["whatsapp"]
        texto = campo.text().replace("(", "").replace(")", "").replace("-", "").replace(" ", "")
        if len(texto) > 2:
            texto = f"({texto[:2]}) {texto[2:]}"
        if len(texto) > 10:
            texto = f"{texto[:10]}-{texto[10:]}"
        campo.setText(texto)
        campo.setCursorPosition(len(texto))  # Mantém o cursor no final

    def formatar_cep(self):
        """Formata automaticamente o campo CEP."""
        campo = self.campos["cep"]
        texto = campo.text().replace("-", "")
        if len(texto) > 5:
            texto = f"{texto[:5]}-{texto[5:]}"
        campo.setText(texto)
        campo.setCursorPosition(len(texto))  # Mantém o cursor no final

    def validar_email(self):
        """Valida o formato do e-mail."""
        email = self.campos["email"].text().strip()
        if email and not re.match(r"[^@]+@[^@]+\.[^@]+", email):
            QMessageBox.warning(self, "Atenção", "Formato de e-mail inválido.")
            self.campos["email"].clear()

    def criar_tabela(self):
        """Cria a tabela no banco de dados se não existir."""
        self.cursor.execute("""
        CREATE TABLE IF NOT EXISTS cadastros (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            codigo INTEGER UNIQUE,
            nome TEXT,
            cpf_cnpj TEXT UNIQUE,
            whatsapp TEXT,
            email TEXT,
            endereco TEXT,
            numero TEXT,
            cep TEXT,
            bairro TEXT,
            cidade TEXT
        )
        """)
        self.conn.commit()

    def gerar_codigo(self):
        """Gera um código numérico sequencial."""
        self.cursor.execute("SELECT MAX(codigo) FROM cadastros")
        ultimo_codigo = self.cursor.fetchone()[0]
        return (ultimo_codigo + 1) if ultimo_codigo is not None else 1

    def carregar_dados(self):
        """Carrega os dados salvos na tabela."""
        self.tabela.setRowCount(0)
        self.cursor.execute(
            "SELECT id, codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade FROM cadastros"
        )
        for row_idx, row_data in enumerate(self.cursor.fetchall()):
            self.tabela.insertRow(row_idx)
            # Inserir os dados na ordem correta
            colunas = [
                str(row_data[1]),  # CÓD
                str(row_data[2]),  # NOME/RAZÃO
                str(row_data[3]),  # CPF/CNPJ
                str(row_data[4]),  # WHATSAPP
                str(row_data[5]),  # EMAIL
                str(row_data[7]),  # ENDEREÇO
                str(row_data[8]),  # Nº
                str(row_data[6]),  # CEP
                str(row_data[9]),  # BAIRRO
                str(row_data[10])  # CIDADE
            ]
            for col_idx, value in enumerate(colunas):
                self.tabela.setItem(row_idx, col_idx, QTableWidgetItem(value))
                self.tabela.resizeColumnsToContents()

    def buscar_e_preencher(self):
        """Busca os dados e, se encontrar por código, CPF, CNPJ preenche o formulário."""
        termo = self.campo_pesquisa.text().strip()
        if not termo:
            self.carregar_dados()
            return

        # Verifica se o termo é numérico
        if termo.isdigit():
        # Determina o tipo de pesquisa com base no tamanho do termo
            if len(termo) == 11:  # CPF
                self.buscar_por_cpf_cnpj(termo)
            elif len(termo) == 14:  # CNPJ
                self.buscar_por_cpf_cnpj(termo)
            elif 10 <= len(termo) <= 11:  # WhatsApp
                self.buscar_por_whatsapp(termo)
            else:  # Código
                self.preencher_formulario_por_codigo(termo)
        else:
        # Pesquisa por nome ou outros campos com texto
            self.buscar_dados(termo)
            
    def buscar_por_cpf_cnpj(self, termo):
        """Busca um registro pelo CPF ou CNPJ."""
        self.cursor.execute(
            "SELECT id, codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade FROM cadastros WHERE REPLACE(REPLACE(REPLACE(cpf_cnpj, '.', ''), '-', ''), '/', '') = ?",
            (termo,)
        )
        resultado = self.cursor.fetchone()
        if resultado:
            self.exibir_resultado_na_tabela(resultado)
        else:
            QMessageBox.warning(self, "Atenção", f"Nenhum cadastro encontrado com o CPF/CNPJ: {termo}")
    
    def buscar_por_whatsapp(self, termo):
        """Busca um registro pelo WhatsApp."""
        self.cursor.execute(
            "SELECT id, codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade FROM cadastros WHERE REPLACE(REPLACE(REPLACE(whatsapp, '(', ''), ')', ''), '-', '') = ?",
            (termo,)
        )
        resultado = self.cursor.fetchone()
        if resultado:
            self.exibir_resultado_na_tabela(resultado)
        else:
            QMessageBox.warning(self, "Atenção", f"Nenhum cadastro encontrado com o WhatsApp: {termo}")
            
    def exibir_resultado_na_tabela(self, resultado):
        """Exibe o resultado da pesquisa na tabela e no formulário."""
        self.tabela.setRowCount(0)  # Limpa a tabela
        row_data = resultado
        self.tabela.insertRow(0)
        for col_idx, value in enumerate(row_data):
            self.tabela.setItem(0, col_idx, QTableWidgetItem(str(value)))
            self.tabela.resizeColumnsToContents()

    # Preenche o formulário com os dados encontrados
            dados = {
                "nome": row_data[2],
                "cpf_cnpj": row_data[3],
                "whatsapp": row_data[4],
                "email": row_data[5],
                "cep": row_data[6],
                "endereco": row_data[7],
                "numero": row_data[8],
                "bairro": row_data[9],
                "cidade": row_data[10],
            }
            for campo, valor in dados.items():
                self.campos[campo].setText(valor)

    def buscar_dados(self, termo):
        """Filtra os dados na tabela com base no termo de pesquisa."""
        self.tabela.setRowCount(0)

        # Verifica se o termo é numérico (para CPF, CNPJ)
        if termo.isdigit():
            termo_limpo = termo  # Mantém o termo original, já que é numérico
        else:
            termo_limpo = None  # Não aplica limpeza para nomes

            # Consulta SQL com remoção de formatação nos campos cpf_cnpj e whatsapp
            if termo_limpo:  # Se o termo for numérico
                self.cursor.execute(
                    """
                    SELECT codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade
                    FROM cadastros
                    WHERE 
                    REPLACE(REPLACE(REPLACE(cpf_cnpj, '.', ''), '-', ''), '/', '') LIKE ?
                    OR REPLACE(REPLACE(REPLACE(whatsapp, '(', ''), ')', ''), '-', '') LIKE ?
                    """,
                    (f"%{termo_limpo}%", f"%{termo_limpo}%"),
                )
            else:  # Se o termo for textual (nome)
                self.cursor.execute(
                    """
                    SELECT codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade
                    FROM cadastros
                    WHERE nome LIKE ?
                    """,
                    (f"%{termo}%",),
                )

            # Preenche a tabela com os resultados da consulta
                for row_idx, row_data in enumerate(self.cursor.fetchall()):
                    self.tabela.insertRow(row_idx)
                    for col_idx, value in enumerate(row_data):
                        self.tabela.setItem(row_idx, col_idx, QTableWidgetItem(str(value)))
                        self.tabela.resizeColumnsToContents()

    def preencher_formulario_por_codigo(self, codigo):
        """Busca um registro por código e preenche o formulário."""
        self.cursor.execute(
            "SELECT nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade FROM cadastros WHERE codigo=?",
            (codigo,),
        )
        registro = self.cursor.fetchone()
        if registro:
            dados = {
                "nome": registro[0],
                "cpf_cnpj": registro[1],
                "whatsapp": registro[2],
                "email": registro[3],
                "cep": registro[4],
                "endereco": registro[5],
                "numero": registro[6],
                "bairro": registro[7],
                "cidade": registro[8],
            }
            for campo, valor in dados.items():
                self.campos[campo].setText(valor)
            # Selecionar a linha correspondente na tabela (opcional)
            for row in range(self.tabela.rowCount()):
                if self.tabela.item(row, 1).text() == codigo:  # A coluna do código é a segunda (índice 1)
                    self.tabela.selectRow(row)
                    break
            self.registro_para_alterar = codigo  # Define para possível alteração
        else:
            QMessageBox.warning(self, "Atenção", f"Nenhum cadastro encontrado com o código: {codigo}")
            self.limpar_campos_e_deselecionar()

    def limpar_campos_para_cadastro(self):
        """Limpa os campos para um novo cadastro e deseleciona a tabela."""
        for campo in self.campos.values():
            campo.clear()
        self.tabela.clearSelection()
        self.registro_para_alterar = None

    def limpar_campos_e_deselecionar(self):
        """Limpa os campos do formulário, o campo de pesquisa e deseleciona qualquer item na tabela."""
        # Limpa os campos do formulário
        for campo in self.campos.values():
            campo.clear()
    
            # Limpa o campo de pesquisa
            self.campo_pesquisa.clear()
    
            # Deseleciona qualquer item na tabela
            self.tabela.clearSelection()
    
            # Reseta a variável de controle para alteração
            self.registro_para_alterar = None

    def salvar_dados(self):
        """Salva os dados no banco de dados ou atualiza um registro existente."""
        nome = self.campos["nome"].text().strip()
        cpf_cnpj = self.campos["cpf_cnpj"].text().strip()
        whatsapp = self.campos["whatsapp"].text().strip()
        email = self.campos["email"].text().strip()
        cep = self.campos["cep"].text().strip()
        endereco = self.campos["endereco"].text().strip()
        numero = self.campos["numero"].text().strip()
        bairro = self.campos["bairro"].text().strip()
        cidade = self.campos["cidade"].text().strip()

        if not nome or not cpf_cnpj:
            QMessageBox.warning(self, "Atenção", "Nome/Razão e CPF/CNPJ são obrigatórios.")
            return

        dados = (nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade)

        if self.registro_para_alterar:
            try:
                self.cursor.execute(
                    """
                    UPDATE cadastros SET nome=?, cpf_cnpj=?, whatsapp=?, email=?,
                    cep=?, endereco=?, numero=?, bairro=?, cidade=?
                    WHERE codigo=?
                    """,
                    (*dados, self.registro_para_alterar),
                )
                self.conn.commit()
                QMessageBox.information(self, "Sucesso", "Dados atualizados com sucesso!")
            except sqlite3.IntegrityError:
                QMessageBox.critical(self, "Erro", "CPF/CNPJ já cadastrado.")
                return
            except Exception as e:
                QMessageBox.critical(self, "Erro", f"Erro ao atualizar dados: {e}")
        else:
            codigo = self.gerar_codigo()
            try:
                self.cursor.execute(
                    """
                    INSERT INTO cadastros (codigo, nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade)
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
                    """,
                    (codigo, *dados),
                )
                self.conn.commit()
                QMessageBox.information(self, "Sucesso", "Dados salvos com sucesso!")
            except sqlite3.IntegrityError:
                QMessageBox.critical(self, "Erro", "CPF/CNPJ já cadastrado.")
                return
            except Exception as e:
                QMessageBox.critical(self, "Erro", f"Erro ao salvar dados: {e}")

                self.carregar_dados()
                self.limpar_campos_e_deselecionar()
        
        # Feedback visual
        for campo in self.campos.values():
            campo.setStyleSheet("background-color: #d4edda; border: 1px solid #c3e6cb;")  # Verde claro
            QTimer.singleShot(2000, lambda: self.resetar_estilos())  # Reseta o estilo após 2 segundos

    def resetar_estilos(self):
        for campo in self.campos.values():
            campo.setStyleSheet("")  # Remove o estilo personalizado

    def alterar_dados(self):
        """Prepara para alterar os dados de um registro selecionado."""
        selected_row = self.tabela.currentRow()
        if selected_row == -1:
            QMessageBox.warning(self, "Erro", "Selecione um registro para alterar.")
            return
        self.registro_para_alterar = self.tabela.item(selected_row, 1).text()  # Pega o código
        self.carregar_registro_selecionado(self.tabela.item(selected_row, 0))  # Carrega os dados no formulário

    def apagar_dados(self):
        """Apaga um registro selecionado."""
        selected_row = self.tabela.currentRow()
        if selected_row == -1:
            QMessageBox.warning(self, "Erro", "Selecione um registro para apagar.")
            return
        codigo = self.tabela.item(selected_row, 0).text()  # Pega o código da coluna correta
        confirmacao = QMessageBox.question(
                self, "Confirmação", f"Deseja realmente apagar o registro com código: {codigo}?", QMessageBox.Yes | QMessageBox.No
            )
        if confirmacao == QMessageBox.Yes:
            try:
                self.cursor.execute("DELETE FROM cadastros WHERE codigo=?", (codigo,))
                self.conn.commit()
                self.carregar_dados()
                QMessageBox.information(self, "Sucesso", "Registro apagado com sucesso!")
                self.limpar_campos_e_deselecionar()
            except Exception as e:
                QMessageBox.critical(self, "Erro", f"Erro ao apagar registro: {e}")

    def carregar_registro_selecionado(self, item):
        """Carrega os dados de um registro selecionado na tabela para o formulário."""
        if item:
            selected_row = item.row()
            codigo = self.tabela.item(selected_row, 0).text()  # Pega o código da coluna correta
            self.cursor.execute(
                "SELECT nome, cpf_cnpj, whatsapp, email, cep, endereco, numero, bairro, cidade FROM cadastros WHERE codigo=?",
                (codigo,),
            )
        registro = self.cursor.fetchone()
        if registro:
            dados = {
                "nome": registro[0],
                "cpf_cnpj": registro[1],
                "whatsapp": registro[2],
                "email": registro[3],
                "cep": registro[4],
                "endereco": registro[5],
                "numero": registro[6],
                "bairro": registro[7],
                "cidade": registro[8],
            }
            for campo, valor in dados.items():
                self.campos[campo].setText(valor)
            self.registro_para_alterar = codigo

    def preencher_endereco(self):
        """Preenche os campos de endereço automaticamente ao digitar o CEP."""
        cep = self.campos["cep"].text().replace("-", "").strip()
        if len(cep) == 8:
            endereco = self.buscar_endereco_api(cep)
            if endereco:
                self.campos["endereco"].setText(endereco["logradouro"])
                self.campos["bairro"].setText(endereco["bairro"])
                self.campos["cidade"].setText(endereco["localidade"])
            else:
                QMessageBox.warning(self, "Erro", "CEP não encontrado.")
        elif len(cep) > 0:
            QMessageBox.warning(self, "Atenção", "CEP deve ter 8 dígitos.")

    def buscar_endereco_api(self, cep):
        """Busca o endereço usando a API ViaCEP."""
        url = f"https://viacep.com.br/ws/{cep}/json/"
        try:
            response = requests.get(url)
            response.raise_for_status()  # Levanta uma exceção para erros HTTP
            data = response.json()
            if not data.get("erro"):
                return data
            return None
        except requests.exceptions.RequestException as e:
            QMessageBox.critical(self, "Erro de Rede", f"Erro ao buscar CEP: {e}")
            return None

    def closeEvent(self, event):
        """Fecha a conexão com o banco de dados ao fechar o aplicativo."""
        self.conn.close()
        event.accept()


    if __name__ == "__main__":
      app = QApplication(sys.argv)
      window = CadastroApp()
      window.show()
      sys.exit(app.exec_())
