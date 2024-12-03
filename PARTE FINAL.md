#Matheus Nogueira Diniz Amorim 
#Silvania Alves Oliveira 
#pip install tabulate
#pip install psycopg2

import psycopg2 # type: ignore
import datetime
# Função para conectar ao banco de dados
def connect_db():
    try:
        conn = psycopg2.connect(
            dbname="TESTE",
            user="postgres",
            password="760902",
            host="127.0.0.1",
            port="5432"
        )
        return conn
    except Exception as e:
        print("Erro ao conectar ao banco!", e)
        exit(1)
from tabulate import tabulate
# Função para exibir dados de uma tabela
def display_table(cursor, table_name):
    valid_tables = ["Estado", "Cidade", "Funcionario", "Cliente", "Pessoa_Juridica", "Pessoa_Fisica", "Frete"]
    if table_name not in valid_tables:
        print("Nome de tabela inválido!")
        return
    try:
        cursor.execute(f"SELECT * FROM {table_name}")
        rows = cursor.fetchall()
        columns = [desc[0] for desc in cursor.description]
        if rows:
            print(f"\nDados da tabela {table_name}:")
            print(tabulate(rows, headers=columns, tablefmt="fancy_grid"))
        else:
            print(f"\nA tabela {table_name} está vazia.")
    except Exception as e:
        print(f"Erro ao exibir tabela {table_name}: {e}")
# Validações auxiliares
def validar_data(data_str):
    try:
        datetime.datetime.strptime(data_str, "%Y-%m-%d")
        return True
    except ValueError:
        return False
def validar_uf(uf):
    ufs_validas = ["AC", "AL", "AP", "AM", "BA", "CE", "DF", "ES", "GO", "MA", 
                   "MT", "MS", "MG", "PA", "PB", "PR", "PE", "PI", "RJ", "RN", 
                   "RS", "RO", "RR", "SC", "SP", "SE", "TO"]
    return uf.upper() in ufs_validas
def validar_float(valor):
    try:
        float(valor)
        return True
    except ValueError:
        return False
def validar_inteiro(valor):
    try:
        int(valor)
        return True
    except ValueError:
        return False
# CRUD para a tabela Estado
def crud_estado(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Estado ---")
        print("1. Inserir Estado")
        print("2. Atualizar Estado")
        print("3. Deletar Estado")
        print("4. Listar Estados")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção! ")
        if choice == "1":
            uf = input("Digite a UF: ")
            while not validar_uf(uf):
               print("UF inválida!")
               uf = input("Digite a UF: ")
            nome = input("Digite o nome do estado: ")
            icms_local = input("Digite o ICMS local: ")
            while not validar_float(icms_local):
               print("Valor de ICMS local deve ser numérico!")
               icms_local = input("Digite o ICMS local: ")
            icms_outro_local = input("Digite o ICMS de outro local: ")
            while not validar_float(icms_outro_local):
               print("Valor de ICMS de outro local deve ser numérico!")
               icms_outro_local = input("Digite o ICMS de outro local: ")
            try:
               cursor.execute(
                     "INSERT INTO Estado (UF, NOME_ESTADO, ICMS_LOCAL, ICMS_OUTRO_LOCAL) VALUES (%s, %s, %s, %s)",
                     (uf, nome, float(icms_local), float(icms_outro_local))
               )
               conn.commit()
               print("Estado inserido com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao inserir estado!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente a UF até que seja válido e exista no banco
               uf = input("Digite a UF do estado que deseja atualizar: ")
               if not validar_uf(uf):
                     print("UF inválida!")
                     continue
               try:
                     # Verifica se a UF existe na tabela Estado
                     cursor.execute("SELECT 1 FROM Estado WHERE UF = %s", (uf,))
                     if cursor.fetchone() is None:
                        print("UF não encontrada no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar UF!", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação da UF, continua com a atualização
            nome = input("Digite o novo nome do estado: ")
            while True:
               try:
                     icms_local = float(input("Digite o novo ICMS local: "))
                     break
               except ValueError:
                     print("Valor inválido, O ICMS local deve ser numérico!")
            while True:
               try:
                     icms_outro_local = float(input("Digite o novo ICMS de outro local: "))
                     break
               except ValueError:
                     print("Valor inválido, O ICMS de outro local deve ser numérico")
            try:
               cursor.execute(
                     "UPDATE Estado SET NOME_ESTADO = %s, ICMS_LOCAL = %s, ICMS_OUTRO_LOCAL = %s WHERE UF = %s",
                     (nome, icms_local, icms_outro_local, uf)
               )
               conn.commit()
               print("Estado atualizado com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar estado!", e)
        elif choice == "3":
            while True:  # Loop para pedir repetidamente a UF até que seja válida e exista no banco
               uf = input("Digite a UF do estado que deseja deletar: ")
               if not validar_uf(uf):
                     print("UF inválida!")
                     continue
               try:
                     # Verifica se a UF existe na tabela Estado
                     cursor.execute("SELECT 1 FROM Estado WHERE UF = %s", (uf,))
                     if cursor.fetchone() is None:
                        print("UF não encontrada no banco de dados!")
                     else:
                        # Se a UF for válida e existir no banco, prossegue com a deleção
                        cursor.execute("DELETE FROM Estado WHERE UF = %s", (uf,))
                        conn.commit()
                        print("Estado deletado com sucesso!")
                        break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar estado!", e)
                     break
        elif choice == "4":
            display_table(cursor, "Estado")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Cidade
def crud_cidade(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Cidade ---")
        print("1. Inserir Cidade")
        print("2. Atualizar Cidade")
        print("3. Deletar Cidade")
        print("4. Listar Cidades")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
            nome = input("Digite o nome da cidade: ")
            # Validação para preço por peso
            preco_peso = input("Digite o preço por peso: ")
            while not validar_float(preco_peso):
               print("O preço por peso deve ser numérico!")
               preco_peso = input("Digite o preço por peso: ")
            # Validação para preço por valor
            preco_valor = input("Digite o preço por valor: ")
            while not validar_float(preco_valor):
               print("O preço por valor deve ser numérico!")
               preco_valor = input("Digite o preço por valor: ")
            uf = input("Digite a UF associada: ")
            # Validação para UF
            while not validar_uf(uf):
               print("UF inválida! Deve conter 2 letras (ex: SP, RJ).")
               uf = input("Digite a UF associada: ")
            try:
               # Verificar se a UF existe no banco
               cursor.execute("SELECT UF FROM Estado WHERE UF = %s", (uf,))
               result = cursor.fetchone()
               while not result:
                     print("UF não encontrada no banco de dados!")
                     uf = input("Digite a UF associada: ")
                     cursor.execute("SELECT UF FROM Estado WHERE UF = %s", (uf,))
                     result = cursor.fetchone()
               # Verificar se a cidade já existe no banco
               cursor.execute("SELECT NOME_CIDADE FROM Cidade WHERE NOME_CIDADE = %s AND UF = %s", (nome, uf))
               cidade_existe = cursor.fetchone()
               if cidade_existe:
                     print(f"A cidade '{nome}' já existe no banco de dados para a UF '{uf}'.")
               else:
                     # Inserir a nova cidade
                     cursor.execute(
                        "INSERT INTO Cidade (NOME_CIDADE, PRECO_UNIT_PESO, PRECO_UNIT_VALOR, UF) VALUES (%s, %s, %s, %s)",
                        (nome, float(preco_peso), float(preco_valor), uf)
                     )
                     conn.commit()
                     print("Cidade inserida com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao inserir cidade!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente o nome da cidade até que seja válido e exista no banco
               nome = input("Digite o nome da cidade que deseja atualizar: ")
               if not nome.strip():
                     print("Nome da cidade não pode estar vazio!")
                     continue
               try:
                     # Verifica se o nome da cidade existe na tabela Cidade
                     cursor.execute("SELECT 1 FROM Cidade WHERE NOME_CIDADE = %s", (nome,))
                     if cursor.fetchone() is None:
                        print("Cidade não encontrada no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar o nome da cidade!", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação do nome da cidade, continua com a atualização
            preco_peso = input("Digite o novo preço por peso: ")
            while not validar_float(preco_peso):
               print("O preço por peso deve ser numérico!")
               preco_peso = input("Digite o novo preço por peso: ")
            preco_valor = input("Digite o novo preço por valor: ")
            while not validar_float(preco_valor):
               print("O preço por valor deve ser numérico!")
               preco_valor = input("Digite o novo preço por valor: ")
            # Validação da UF
            while True:
               uf = input("Digite a nova UF associada: ")
               if not validar_uf(uf):
                     print("UF inválida!")
                     continue
               try:
                     # Verificar se a UF está no banco de dados
                     cursor.execute("SELECT 1 FROM Estado WHERE UF = %s", (uf,))
                     if cursor.fetchone() is None:
                        print("UF não encontrada no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar a UF no banco de dados!", e)
                     return  # Sai da função em caso de erro
            # Atualiza a cidade com os novos dados
            try:
               cursor.execute(
                     "UPDATE Cidade SET PRECO_UNIT_PESO = %s, PRECO_UNIT_VALOR = %s, UF = %s WHERE NOME_CIDADE = %s",
                     (float(preco_peso), float(preco_valor), uf, nome)
               )
               conn.commit()
               print("Cidade atualizada com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar cidade!", e)
        elif choice == "3":
            while True:
               nome_cidade = input("Digite o nome da cidade que deseja deletar: ")
               if not nome_cidade:
                     print("O nome da cidade não pode estar vazio!")
                     continue
               try:
                     cursor.execute("SELECT COUNT(*) FROM Cidade WHERE NOME_CIDADE = %s", (nome_cidade,))
                     resultado = cursor.fetchone()
                     if resultado[0] == 0:
                        print("Cidade não encontrada no banco de dados!")
                        continue
                     cursor.execute("DELETE FROM Cidade WHERE NOME_CIDADE = %s", (nome_cidade,))
                     conn.commit()
                     print("Cidade deletada com sucesso!")
                     break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar cidade!", e)
        elif choice == "4":
            display_table(cursor, "Cidade")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Funcionário
def crud_funcionario(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Funcionário ---")
        print("1. Inserir Funcionário")
        print("2. Atualizar Funcionário")
        print("3. Deletar Funcionário")
        print("4. Listar Funcionários")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
            nome = input("Digite o nome do funcionário: ")
            try:
               # Verifica se o funcionário já existe no banco de dados
               cursor.execute(
                     "SELECT COUNT(*) FROM Funcionario WHERE NOME_FUNCIONARIO = %s",
                     (nome,)
               )
               resultado = cursor.fetchone()
               if resultado[0] > 0:
                     print("Funcionário já existe no banco de dados!")
               else:
                     # Insere o novo funcionário
                     cursor.execute(
                        "INSERT INTO Funcionario (NOME_FUNCIONARIO) VALUES (%s)",
                        (nome,)
                     )
                     conn.commit()
                     print("Funcionário inserido com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao inserir funcionário!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente o número de registro até que seja válido e exista no banco
               num_registro = input("Digite o número de registro do funcionário que deseja atualizar: ")
               if not validar_inteiro(num_registro):
                     print("O número de registro deve ser um número inteiro!")
                     continue
               try:
                     # Verifica se o número de registro do funcionário existe na tabela Funcionario
                     cursor.execute("SELECT 1 FROM Funcionario WHERE NUM_REGISTRO = %s", (int(num_registro),))
                     if cursor.fetchone() is None:
                        print("Número de registro não encontrado no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar o número de registro!", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação do número de registro do funcionário, continua com a atualização
            nome = input("Digite o novo nome do funcionário: ")
            try:
               cursor.execute(
                     "UPDATE Funcionario SET NOME_FUNCIONARIO = %s WHERE NUM_REGISTRO = %s",
                     (nome, int(num_registro))
               )
               conn.commit()
               print("Funcionário atualizado com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar funcionário!", e)
        elif choice == "3":
            while True:
               num_registro = input("Digite o número de registro do funcionário que deseja deletar: ")
               if not validar_inteiro(num_registro):
                     print("O número de registro deve ser um número inteiro!")
                     continue
               try:
                     cursor.execute("SELECT COUNT(*) FROM Funcionario WHERE NUM_REGISTRO = %s", (int(num_registro),))
                     resultado = cursor.fetchone()
                     if resultado[0] == 0:
                        print("Funcionário não encontrado no banco de dados!")
                        continue
                     cursor.execute("DELETE FROM Funcionario WHERE NUM_REGISTRO = %s", (int(num_registro),))
                     conn.commit()
                     print("Funcionário deletado com sucesso!")
                     break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar funcionário!", e)
        elif choice == "4":
            display_table(cursor, "Funcionario")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Cliente
def crud_cliente(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Cliente ---")
        print("1. Inserir Cliente")
        print("2. Atualizar Cliente")
        print("3. Deletar Cliente")
        print("4. Listar Clientes")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
         data_insc = input("Digite a data de inscrição (YYYY-MM-DD): ")
         while not validar_data(data_insc):
            print("Data inválida! Use o formato YYYY-MM-DD.")
            data_insc = input("Digite a data de inscrição (YYYY-MM-DD): ")
         endereco = input("Digite o endereço: ")
         telefone = input("Digite o telefone: ")
         try:
            cursor.execute(
                  "INSERT INTO Cliente (DATA_INSC, ENDERECO, TELEFONE) VALUES (%s, %s, %s)",
                  (data_insc, endereco, telefone)
            )
            conn.commit()
            print("Cliente inserido com sucesso!")
         except Exception as e:
            conn.rollback()
            print("Erro ao inserir cliente!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente o código do cliente até que seja válido e exista no banco
               cod_cliente = input("Digite o código do cliente que deseja atualizar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     # Verifica se o código do cliente existe na tabela Cliente
                     cursor.execute("SELECT 1 FROM Cliente WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     if cursor.fetchone() is None:
                        print("Cliente não encontrado no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar o código do cliente:", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação do código do cliente, continua com a atualização
            data_insc = input("Digite a nova data de inscrição (YYYY-MM-DD): ")
            endereco = input("Digite o novo endereço: ")
            telefone = input("Digite o novo telefone: ")
            if not validar_data(data_insc):
               print("Data inválida! Use o formato YYYY-MM-DD.")
               return
            try:
               cursor.execute(
                     "UPDATE Cliente SET DATA_INSC = %s, ENDERECO = %s, TELEFONE = %s WHERE COD_CLIENTE = %s",
                     (data_insc, endereco, telefone, int(cod_cliente))
               )
               conn.commit()
               print("Cliente atualizado com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar cliente!", e)
        elif choice == "3":
            while True:
               cod_cliente = input("Digite o código do cliente que deseja deletar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     cursor.execute("SELECT COUNT(*) FROM Cliente WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     resultado = cursor.fetchone()
                     if resultado[0] == 0:
                        print("Cliente não encontrado na base de dados!")
                        continue
                     cursor.execute("DELETE FROM Cliente WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     conn.commit()
                     print("Cliente deletado com sucesso!")
                     break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar cliente!", e)
        elif choice == "4":
            display_table(cursor, "Cliente")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Pessoa Jurídica
def crud_pessoa_juridica(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Pessoa Jurídica ---")
        print("1. Inserir Pessoa Jurídica")
        print("2. Atualizar Pessoa Jurídica")
        print("3. Deletar Pessoa Jurídica")
        print("4. Listar Pessoas Jurídicas")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
            cod_cliente = input("Digite o código do cliente associado: ")
            razao_social = input("Digite a razão social: ")
            insc_estadual = input("Digite a inscrição estadual: ")
            cnpj = input("Digite o CNPJ: ")
            if not validar_inteiro(cod_cliente):
                print("O código do cliente deve ser um número inteiro!")
                continue
            try:
                cursor.execute(
                    "INSERT INTO Pessoa_Juridica (COD_CLIENTE, RAZAO_SOCIAL, INSC_ESTADUAL, CNPJ) VALUES (%s, %s, %s, %s)",
                    (int(cod_cliente), razao_social, insc_estadual, cnpj)
                )
                conn.commit()
                print("Pessoa Jurídica inserida com sucesso!")
            except Exception as e:
                conn.rollback()
                print("Erro ao inserir pessoa jurídica!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente o código do cliente até que seja válido e exista no banco
               cod_cliente = input("Digite o código da pessoa jurídica que deseja atualizar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     # Verifica se o código do cliente existe na tabela Pessoa_Juridica
                     cursor.execute("SELECT 1 FROM Pessoa_Juridica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     if cursor.fetchone() is None:
                        print("Pessoa jurídica não encontrado no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar o código do cliente:", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação do código do cliente, continua com a atualização
            razao_social = input("Digite a nova razão social: ")
            insc_estadual = input("Digite a nova inscrição estadual: ")
            cnpj = input("Digite o novo CNPJ: ")
            try:
               cursor.execute(
                     "UPDATE Pessoa_Juridica SET RAZAO_SOCIAL = %s, INSC_ESTADUAL = %s, CNPJ = %s WHERE COD_CLIENTE = %s",
                     (razao_social, insc_estadual, cnpj, int(cod_cliente))
               )
               conn.commit()
               print("Pessoa Jurídica atualizada com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar pessoa jurídica:", e)
        elif choice == "3":
            while True:
               cod_cliente = input("Digite o código da pessoa jurídica que deseja deletar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     cursor.execute("SELECT COUNT(*) FROM Pessoa_Juridica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     resultado = cursor.fetchone()
                     if resultado[0] == 0:
                        print("Pessoa jurídica não encontrada no banco de dados!")
                        continue
                     cursor.execute("DELETE FROM Pessoa_Juridica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     conn.commit()
                     print("Pessoa Jurídica deletada com sucesso!")
                     break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar pessoa jurídica!", e)
        elif choice == "4":
            display_table(cursor, "Pessoa_Juridica")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Pessoa Física
def crud_pessoa_fisica(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Pessoa Física ---")
        print("1. Inserir Pessoa Física")
        print("2. Atualizar Pessoa Física")
        print("3. Deletar Pessoa Física")
        print("4. Listar Pessoas Físicas")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
            cod_cliente = input("Digite o código do cliente associado: ")
            nome_cliente = input("Digite o nome do cliente: ")
            cpf = input("Digite o CPF: ")
            if not validar_inteiro(cod_cliente):
                print("O código do cliente deve ser um número inteiro!")
                continue
            try:
                cursor.execute(
                    "INSERT INTO Pessoa_Fisica (COD_CLIENTE, NOME_CLIENTE, CPF) VALUES (%s, %s, %s)",
                    (int(cod_cliente), nome_cliente, cpf)
                )
                conn.commit()
                print("Pessoa Física inserida com sucesso!")
            except Exception as e:
                conn.rollback()
                print("Erro ao inserir pessoa física!", e)
        elif choice == "2":
            while True:  # Loop para pedir repetidamente o código do cliente até que seja válido e exista no banco
               cod_cliente = input("Digite o código do cliente que deseja atualizar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     # Verifica se o código do cliente existe na tabela Pessoa_Fisica
                     cursor.execute("SELECT 1 FROM Pessoa_Fisica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     if cursor.fetchone() is None:
                        print("Pessoa física não encontrado no banco de dados!")
                     else:
                        break
               except Exception as e:
                     print("Erro ao verificar o código do cliente:", e)
                     return  # Sai da função se houver algum erro no banco de dados
            # Após a validação do código do cliente, continua com a atualização
            nome_cliente = input("Digite o novo nome do cliente: ")
            cpf = input("Digite o novo CPF: ")
            try:
               cursor.execute(
                     "UPDATE Pessoa_Fisica SET NOME_CLIENTE = %s, CPF = %s WHERE COD_CLIENTE = %s",
                     (nome_cliente, cpf, int(cod_cliente))
               )
               conn.commit()
               print("Pessoa física atualizada com sucesso!")
            except Exception as e:
               conn.rollback()
               print("Erro ao atualizar pessoa física!", e)
        elif choice == "3":
            while True:
               cod_cliente = input("Digite o código da pessoa física que deseja deletar: ")
               if not validar_inteiro(cod_cliente):
                     print("O código do cliente deve ser um número inteiro!")
                     continue
               try:
                     cursor.execute("SELECT COUNT(*) FROM Pessoa_Fisica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     resultado = cursor.fetchone()
                     if resultado[0] == 0:
                        print("Pessoa física não encontrada no banco de dados!")
                        continue
                     cursor.execute("DELETE FROM Pessoa_Fisica WHERE COD_CLIENTE = %s", (int(cod_cliente),))
                     conn.commit()
                     print("Pessoa Física deletada com sucesso!")
                     break
               except Exception as e:
                     conn.rollback()
                     print("Erro ao deletar pessoa física!", e)
        elif choice == "4":
            display_table(cursor, "Pessoa_Fisica")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
# CRUD para a tabela Frete
def crud_frete(cursor, conn):
    while True:
        print("\n--- CRUD Tabela Frete ---")
        print("1. Inserir Frete")
        print("2. Atualizar Frete")
        print("3. Deletar Frete")
        print("4. Listar Fretes")
        print("0. Voltar ao menu principal")
        choice = input("Escolha uma opção: ")
        if choice == "1":
            quem_paga = input("Quem paga (Remetente/Destinatário): ")
            while True:
                peso_ou_valor = input("Cobrança por Peso ou Valor (P/V): ")
                if peso_ou_valor in ['P', 'V']:
                    break
                else:
                    print("Opção inválida para cobrança! Deve ser 'P' ou 'V'.")
            peso = None
            valor = None
            if peso_ou_valor == 'P':
                while True:
                    peso = input("Digite o peso: ")
                    if validar_float(peso):
                        peso = float(peso)
                        break
                    else:
                        print("Peso deve ser numérico!")
            elif peso_ou_valor == 'V':
                while True:
                    valor = input("Digite o valor: ")
                    if validar_float(valor):
                        valor = float(valor)
                        break
                    else:
                        print("Valor deve ser numérico!")
            while True:
                icms = input("Digite o ICMS: ")
                if validar_float(icms):
                    icms = float(icms)
                    break
                else:
                    print("ICMS deve ser um valor numérico!")
            while True:
                data_frete = input("Digite a data do frete (YYYY-MM-DD): ")
                if validar_data(data_frete):
                    break
                else:
                    print("Data inválida! Use o formato YYYY-MM-DD.")
            while True:
                pedagio = input("Digite o pedágio: ")
                if validar_float(pedagio):
                    pedagio = float(pedagio)
                    break
                else:
                    print("Pedágio deve ser um valor numérico!")
            while True:
                origem = input("Digite o código da cidade de origem: ")
                if validar_inteiro(origem):
                    origem = int(origem)
                    cursor.execute("SELECT 1 FROM Cidade WHERE codigo_cidade = %s", (origem,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cidade não encontrada no banco de dados!")
                else:
                    print("Código da cidade de origem deve ser um número inteiro!")
            while True:
                destino = input("Digite o código da cidade de destino: ")
                if validar_inteiro(destino):
                    destino = int(destino)
                    cursor.execute("SELECT 1 FROM Cidade WHERE codigo_cidade = %s", (destino,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cidade não encontrada no banco de dados!")
                else:
                    print("Código da cidade de destino deve ser um número inteiro!")
            while True:
                remetente = input("Digite o código do cliente remetente: ")
                if validar_inteiro(remetente):
                    remetente = int(remetente)
                    cursor.execute("SELECT 1 FROM Cliente WHERE cod_cliente = %s", (remetente,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cliente não encontrado no banco de dados")
                else:
                    print("Código do cliente remetente deve ser um número inteiro!")
            while True:
                destinatario = input("Digite o código do cliente destinatário: ")
                if validar_inteiro(destinatario):
                    destinatario = int(destinatario)
                    cursor.execute("SELECT 1 FROM Cliente WHERE cod_cliente = %s", (destinatario,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cliente não encontrado no banco de dados")
                else:
                    print("Código do cliente destinatário deve ser um número inteiro!")
            while True:
                num_reg_func = input("Digite o número de registro do funcionário: ")
                if validar_inteiro(num_reg_func):
                    num_reg_func = int(num_reg_func)
                    cursor.execute("SELECT 1 FROM Funcionario WHERE num_registro = %s", (num_reg_func,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Funcionário não encontrado no banco de dados")
                else:
                    print("Número de registro do funcionário deve ser um número inteiro!")
            try:
                cursor.execute(
                    """INSERT INTO Frete (QUEM_PAGA, PESO_OU_VALOR, PESO, VALOR, ICMS, DATA_FRETE, PEDAGIO, ORIGEM, DESTINO, REMETENTE, DESTINATARIO, NUM_REG_FUNC)
                    VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)""",
                    (quem_paga, peso_ou_valor, peso, valor, icms, data_frete, pedagio, origem, destino, remetente, destinatario, num_reg_func)
                )
                conn.commit()
                print("Frete inserido com sucesso!")
            except Exception as e:
                conn.rollback()
                print("Erro ao inserir frete!", e)
        elif choice == "2":
            while True:
                num_conhecimento = input("Digite o número do conhecimento do frete que deseja atualizar: ")
                if not validar_inteiro(num_conhecimento):
                    print("O número do conhecimento deve ser um número inteiro!")
                    continue
                try:
                    cursor.execute("SELECT 1 FROM Frete WHERE num_conhecimento = %s", (int(num_conhecimento),))
                    if cursor.fetchone() is None:
                        print("Número do conhecimento não encontrado no banco de dados!")
                    else:
                        num_conhecimento = int(num_conhecimento)
                        break
                except Exception as e:
                    print("Erro ao verificar o número do conhecimento!", e)
                    return
            quem_paga = input("Quem paga (Remetente/Destinatário): ")
            while True:
                peso_ou_valor = input("Cobrança por Peso ou Valor (P/V): ")
                if peso_ou_valor in ['P', 'V']:
                    break
                else:
                    print("Opção inválida para cobrança! Deve ser 'P' ou 'V'.")
            peso = None
            valor = None
            if peso_ou_valor == 'P':
                while True:
                    peso = input("Digite o peso: ")
                    if validar_float(peso):
                        peso = float(peso)
                        break
                    else:
                        print("Peso deve ser um valor numérico!")
            elif peso_ou_valor == 'V':
                while True:
                    valor = input("Digite o valor: ")
                    if validar_float(valor):
                        valor = float(valor)
                        break
                    else:
                        print("Valor deve ser um valor numérico!")
            while True:
                icms = input("Digite o ICMS: ")
                if validar_float(icms):
                    icms = float(icms)
                    break
                else:
                    print("ICMS deve ser um valor numérico!")
            while True:
                data_frete = input("Digite a nova data do frete (YYYY-MM-DD): ")
                if validar_data(data_frete):
                    break
                else:
                    print("Data inválida! Use o formato YYYY-MM-DD.")
            while True:
                pedagio = input("Digite o pedágio: ")
                if validar_float(pedagio):
                    pedagio = float(pedagio)
                    break
                else:
                    print("Pedágio deve ser um valor numérico!")
            while True:
                origem = input("Digite o código da cidade de origem: ")
                if validar_inteiro(origem):
                    origem = int(origem)
                    cursor.execute("SELECT 1 FROM Cidade WHERE codigo_cidade = %s", (origem,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cidade de origem não encontrada no banco de dados!")
                else:
                    print("Código da cidade de origem deve ser um número inteiro!")
            while True:
                destino = input("Digite o código da cidade de destino: ")
                if validar_inteiro(destino):
                    destino = int(destino)
                    cursor.execute("SELECT 1 FROM Cidade WHERE codigo_cidade = %s", (destino,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cidade de destino não encontrada no banco de dados!")
                else:
                    print("Código da cidade de destino deve ser um número inteiro!")
            while True:
                remetente = input("Digite o código do cliente remetente: ")
                if validar_inteiro(remetente):
                    remetente = int(remetente)
                    cursor.execute("SELECT 1 FROM Cliente WHERE cod_cliente = %s", (remetente,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cliente não encontrado no banco de dados!")
                else:
                    print("Código do cliente remetente deve ser um número inteiro!")
            while True:
                destinatario = input("Digite o código do cliente destinatário: ")
                if validar_inteiro(destinatario):
                    destinatario = int(destinatario)
                    cursor.execute("SELECT 1 FROM Cliente WHERE cod_cliente = %s", (destinatario,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Cliente não encontrado no banco de dados!")
                else:
                    print("Código do cliente destinatário deve ser um número inteiro!")
            while True:
                num_reg_func = input("Digite o número de registro do funcionário: ")
                if validar_inteiro(num_reg_func):
                    num_reg_func = int(num_reg_func)
                    cursor.execute("SELECT 1 FROM Funcionario WHERE num_registro = %s", (num_reg_func,))
                    if cursor.fetchone() is not None:
                        break
                    else:
                        print("Funcionário não encontrado no banco de dados!")
                else:
                    print("Número de registro do funcionário deve ser um número inteiro!")
            try:
                cursor.execute(
                    """UPDATE Frete SET QUEM_PAGA = %s, PESO_OU_VALOR = %s, PESO = %s, VALOR = %s, ICMS = %s, DATA_FRETE = %s,
                    PEDAGIO = %s, ORIGEM = %s, DESTINO = %s, REMETENTE = %s, DESTINATARIO = %s, NUM_REG_FUNC = %s
                    WHERE NUM_CONHECIMENTO = %s""",
                    (quem_paga, peso_ou_valor, peso, valor, icms, data_frete, pedagio,
                     origem, destino, remetente, destinatario, num_reg_func, num_conhecimento)
                )
                conn.commit()
                print("Frete atualizado com sucesso!")
            except Exception as e:
                conn.rollback()
                print("Erro ao atualizar frete!", e)
        elif choice == "3":
            while True:
                num_conhecimento = input("Digite o número do conhecimento do frete que deseja deletar: ")
                if not validar_inteiro(num_conhecimento):
                    print("O número do conhecimento deve ser um número inteiro!")
                    continue
                try:
                    cursor.execute("SELECT COUNT(*) FROM Frete WHERE num_conhecimento = %s", (int(num_conhecimento),))
                    resultado = cursor.fetchone()
                    if resultado[0] == 0:
                        print("Frete não encontrado no banco de dados!")
                        continue
                    cursor.execute("DELETE FROM Frete WHERE num_conhecimento = %s", (int(num_conhecimento),))
                    conn.commit()
                    print("Frete deletado com sucesso!")
                    break
                except Exception as e:
                    conn.rollback()
                    print("Erro ao deletar frete!", e)
        elif choice == "4":
            display_table(cursor, "Frete")
        elif choice == "0":
            break
        else:
            print("Opção inválida!")
#/////////////////////////////////////////////////////////////////////////////////////////////////////////////
#PARTE FINAL DO TRABALHO 
def arrecadacao_por_cidade_estado(cursor, estado):
    try:
        query = """
            SELECT
                c.nome_cidade AS cidade,
                e.uf AS estado,
                COUNT(f.num_conhecimento) AS quantidade_fretes,
                SUM(f.valor) AS valor_total_arrecadado
            FROM Frete f
            JOIN Cidade c ON f.destino = c.codigo_cidade
            JOIN Estado e ON c.uf = e.uf
            WHERE e.uf = %s AND EXTRACT(YEAR FROM f.data_frete) = 2024
            GROUP BY c.nome_cidade, e.uf
            ORDER BY valor_total_arrecadado DESC;
        """
        cursor.execute(query, (estado,))
        resultado = cursor.fetchall()
        colunas = ["Cidade", "Estado", "Quantidade de Fretes", "Valor Total Arrecadado"]
        print(tabulate(resultado, headers=colunas, tablefmt="fancy_grid"))
    except Exception as e:
        print("Erro ao calcular arrecadação por cidade/estado:", e)

def media_fretes_por_cidade_estado(cursor, estado):
    try:
        query = """
            SELECT
                c.nome_cidade AS cidade,
                e.uf AS estado,
                AVG(fretes_origem) AS media_fretes_origem,
                AVG(fretes_destino) AS media_fretes_destino
            FROM (
                SELECT
                    origem AS cidade_id,
                    COUNT(*) AS fretes_origem,
                    0 AS fretes_destino
                FROM Frete
                GROUP BY origem
                UNION ALL
                SELECT
                    destino AS cidade_id,
                    0 AS fretes_origem,
                    COUNT(*) AS fretes_destino
                FROM Frete
                GROUP BY destino
            ) subquery
            JOIN Cidade c ON subquery.cidade_id = c.codigo_cidade
            JOIN Estado e ON c.uf = e.uf
            WHERE e.uf = %s
            GROUP BY c.nome_cidade, e.uf;
        """
        cursor.execute(query, (estado,))
        resultado = cursor.fetchall()
        colunas = ["Estado", "Cidade", "Média Fretes Origem", "Média Fretes Destino"]
        print(tabulate(resultado, headers=colunas, tablefmt="fancy_grid"))
    except Exception as e:
        print("Erro ao calcular médias de fretes por cidade/estado:", e)

def fretes_pessoas_juridicas_por_mes(cursor, mes, ano):
    try:
        query = """
            SELECT
                f.num_conhecimento AS num_frete,
                pj.razao_social AS empresa,
                pj.cnpj,
                func.nome_funcionario AS representante
            FROM Frete f
            JOIN Cliente c ON f.remetente = c.cod_cliente OR f.destinatario = c.cod_cliente
            JOIN Pessoa_Juridica pj ON pj.cod_cliente = c.cod_cliente
            JOIN Funcionario func ON f.num_reg_func = func.num_registro
            WHERE EXTRACT(MONTH FROM f.data_frete) = %s
              AND EXTRACT(YEAR FROM f.data_frete) = %s;
        """
        cursor.execute(query, (mes, ano))
        resultado = cursor.fetchall()
        colunas = ["Número do Frete", "Empresa", "CNPJ", "Representante"]
        print(tabulate(resultado, headers=colunas, tablefmt="fancy_grid"))
    except Exception as e:
        print("Erro ao listar fretes de pessoas jurídicas:", e)

# Menu principal
def main():
    conn = connect_db()
    try:
        with conn.cursor() as cursor:
            while True:
                print("\n--- Menu Principal ---")
                print("1. CRUD Estado")
                print("2. CRUD Cidade")
                print("3. CRUD Funcionário")
                print("4. CRUD Cliente")
                print("5. CRUD Pessoa Jurídica")
                print("6. CRUD Pessoa Física")
                print("7. CRUD Frete")
                print("8. Arecadação pro cidade/estado")
                print("9. Media fretes por cidade/estado")
                print("10. Fretes pessoas juridicas por mes")
                print("0. Sair")
                choice = input("Escolha uma opção: ")
                if choice == "1":
                    crud_estado(cursor, conn)
                elif choice == "2":
                    crud_cidade(cursor, conn)
                elif choice == "3":
                    crud_funcionario(cursor, conn)
                elif choice == "4":
                    crud_cliente(cursor, conn)
                elif choice == "5":
                    crud_pessoa_juridica(cursor, conn)
                elif choice == "6":
                    crud_pessoa_fisica(cursor, conn)
                elif choice == "7":
                    crud_frete(cursor, conn)
                elif choice == "8":
                     estado = input("Digite o estado (UF): ")
                     arrecadacao_por_cidade_estado(cursor, estado)
                elif choice == "9":
                     estado = input("Digite o estado (UF): ")
                     media_fretes_por_cidade_estado(cursor, estado)
                elif choice == "10":
                     mes = int(input("Digite o mês (1-12): "))
                     ano = int(input("Digite o ano: "))
                     fretes_pessoas_juridicas_por_mes(cursor, mes, ano)
                elif choice == "0":
                    print("Encerrando o programa...")
                    break
                else:
                    print("Opção inválida!")
    finally:
        conn.close()
if __name__ == "__main__":
    main()