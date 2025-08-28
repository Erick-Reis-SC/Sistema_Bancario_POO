from abc import ABC, abstractmethod
from datetime import datetime
import textwrap

# ----------------------------
# CLASSE HISTÓRICO
# ----------------------------
class Historico:
    def __init__(self):
        self.transacoes = []

    def adicionar_transacao(self, transacao):
        self.transacoes.append(transacao)

    def exibir(self):
        print("\n================ EXTRATO ================")
        if not self.transacoes:
            print("Não foram realizadas movimentações.")
        else:
            for t in self.transacoes:
                print(f"{t['tipo']}:\tR$ {t['valor']:.2f} - {t['data']}")
        print("==========================================")

# ----------------------------
# INTERFACE TRANSAÇÃO
# ----------------------------
class Transacao(ABC):
    @abstractmethod
    def registrar(self, conta):
        pass

# ----------------------------
# CLASSES DE OPERAÇÃO (Depósito e Saque)
# ----------------------------
class Deposito(Transacao):
    def __init__(self, valor):
        self.valor = valor

    def registrar(self, conta):
        if self.valor > 0:
            conta._saldo += self.valor
            conta.historico.adicionar_transacao({
                "tipo": "Depósito",
                "valor": self.valor,
                "data": datetime.now().strftime("%d/%m/%Y %H:%M")
            })
            print("\n=== Depósito realizado com sucesso! ===")
        else:
            print("\n@@@ Operação falhou! Valor inválido. @@@")


class Saque(Transacao):
    def __init__(self, valor):
        self.valor = valor

    def registrar(self, conta):
        if self.valor <= 0:
            print("\n@@@ Operação falhou! Valor inválido. @@@")
            return

        if self.valor > conta._saldo:
            print("\n@@@ Operação falhou! Saldo insuficiente. @@@")
        elif self.valor > conta.limite:
            print("\n@@@ Operação falhou! Valor excede o limite. @@@")
        elif conta.numero_saques >= conta.limite_saques:
            print("\n@@@ Operação falhou! Número máximo de saques excedido. @@@")
        else:
            conta._saldo -= self.valor
            conta.numero_saques += 1
            conta.historico.adicionar_transacao({
                "tipo": "Saque",
                "valor": self.valor,
                "data": datetime.now().strftime("%d/%m/%Y %H:%M")
            })
            print("\n=== Saque realizado com sucesso! ===")

# ----------------------------
# CLASSE CONTA
# ----------------------------
class Conta:
    def __init__(self, numero, cliente):
        self._saldo = 0.0
        self.numero = numero
        self.agencia = "0001"
        self.cliente = cliente
        self.historico = Historico()

    def saldo(self):
        return self._saldo

# ----------------------------
# CLASSE CONTA CORRENTE (Especialização)
# ----------------------------
class ContaCorrente(Conta):
    def __init__(self, numero, cliente, limite=500, limite_saques=3):
        super().__init__(numero, cliente)
        self.limite = limite
        self.limite_saques = limite_saques
        self.numero_saques = 0

# ----------------------------
# CLASSE CLIENTE
# ----------------------------
class Cliente:
    def __init__(self, endereco):
        self.endereco = endereco
        self.contas = []

    def adicionar_conta(self, conta):
        self.contas.append(conta)

    def realizar_transacao(self, conta, transacao):
        transacao.registrar(conta)

# ----------------------------
# SUBCLASSE PESSOA FÍSICA
# ----------------------------
class PessoaFisica(Cliente):
    def __init__(self, nome, cpf, data_nascimento, endereco):
        super().__init__(endereco)
        self.nome = nome
        self.cpf = cpf
        self.data_nascimento = data_nascimento

# ----------------------------
# FUNÇÕES DE INTERFACE (MENU)
# ----------------------------
def menu():
    menu = """\n
    ================ MENU ================
    [d]\tDepositar
    [s]\tSacar
    [e]\tExtrato
    [nc]\tNova conta
    [lc]\tListar contas
    [nu]\tNovo usuário
    [q]\tSair
    => """
    return input(textwrap.dedent(menu))

def main():
    clientes = []
    contas = []
    numero_conta = 1

    while True:
        opcao = menu()

        if opcao == "d":
            cpf = input("Informe o CPF do cliente: ")
            cliente = next((c for c in clientes if c.cpf == cpf), None)
            if not cliente:
                print("\n@@@ Cliente não encontrado! @@@")
                continue

            valor = float(input("Informe o valor do depósito: "))
            conta = cliente.contas[0]
            cliente.realizar_transacao(conta, Deposito(valor))

        elif opcao == "s":
            cpf = input("Informe o CPF do cliente: ")
            cliente = next((c for c in clientes if c.cpf == cpf), None)
            if not cliente:
                print("\n@@@ Cliente não encontrado! @@@")
                continue

            valor = float(input("Informe o valor do saque: "))
            conta = cliente.contas[0]
            cliente.realizar_transacao(conta, Saque(valor))

        elif opcao == "e":
            cpf = input("Informe o CPF do cliente: ")
            cliente = next((c for c in clientes if c.cpf == cpf), None)
            if cliente:
                conta = cliente.contas[0]
                conta.historico.exibir()
                print(f"\nSaldo:\t\tR$ {conta.saldo():.2f}")
            else:
                print("\n@@@ Cliente não encontrado! @@@")


        elif opcao == "nu":
            nome = input("Nome completo: ")
            cpf = input("CPF: ")
            data_nascimento = input("Data de nascimento (dd/mm/aaaa): ")
            endereco = input("Endereço: ")
            cliente = PessoaFisica(nome, cpf, data_nascimento, endereco)
            clientes.append(cliente)
            print("\n=== Cliente criado com sucesso! ===")

        elif opcao == "nc":
            cpf = input("Informe o CPF do cliente: ")
            cliente = next((c for c in clientes if c.cpf == cpf), None)
            if not cliente:
                print("\n@@@ Cliente não encontrado! @@@")
                continue
            conta = ContaCorrente(numero_conta, cliente)
            contas.append(conta)
            cliente.adicionar_conta(conta)
            numero_conta += 1
            print("\n=== Conta criada com sucesso! ===")

        elif opcao == "lc":
            for conta in contas:
                print("=" * 50)
                print(f"Agência: {conta.agencia}\nC/C: {conta.numero}\nTitular: {conta.cliente.nome}")

        elif opcao == "q":
            break

        else:
            print("Operação inválida, tente novamente.")

if __name__ == "__main__":
    main()
