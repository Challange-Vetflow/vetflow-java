# VetFlow API + Web — Challenge FIAP 2026

Solução desenvolvida para o **Challenge FIAP 2026** em parceria com a **CLYVO VET**.

## Integrantes do Grupo

| Nome | RM | Turma |
|------|----|-------|
| Andrei de Paiva Gibbini | 563061 | 2TDSPF |
| Arthur Câmara | 562310 | 2TDSPG |
| Diogo Cunha | 563654 | 2TDSPF |
| Pedro Sakai Silva Zambaca | 565956 | 2TDSPF |
| Pedro Santos Pequini | 561842 | 2TDSPF |

## Problema de Negócio

Tutores de pets só acionam clínicas em urgências ou vacinas óbvias. Isso gera baixa recorrência, menor LTV para as clínicas e histórico clínico fragmentado.

## Solução

Nesta sprint o VetFlow evolui de uma API REST pura para uma **aplicação web completa**: além dos endpoints REST, agora existe uma camada de visualização (Thymeleaf), controle de versões de banco (Flyway) e autenticação/autorização com dois perfis de usuário (Spring Security).

## Arquitetura

```
src/main/java/fiap/com/br/vetflow/
├── config/            SwaggerConfig
├── controller/        API REST — TutorController, PetController, ClinicController,
│                       AppointmentController, VaccineController, MedicationController
├── controller/api/     AuthApiController — login/registro/sessão em JSON, para clientes mobile
├── controller/web/     Frontend — AuthController (login/registro), WebController (dashboard, fluxos)
├── dto/                TutorDtos, PetDtos, ClinicDtos, AppointmentDtos, VaccineDtos, MedicationDtos, RegisterDtos
├── entity/             Tutor, Pet, Clinic, Appointment, Vaccine, Medication, User, UserRole
├── repository/         Interfaces JpaRepository de cada entidade
├── security/           SecurityConfig, VetFlowUserDetailsService
└── service/            Regras de negócio por entidade

src/main/resources/
├── db/migration/       Scripts Flyway (V1, V2, V3)
└── templates/          Views Thymeleaf (auth, dashboard, pets, appointments, vaccines)
```

## Tecnologias

| Camada | Tecnologia |
|---|---|
| Framework | Spring Boot 3.4 |
| Frontend | Thymeleaf + Bootstrap 5 |
| Persistência | Spring Data JPA |
| Migrations | Flyway |
| Segurança | Spring Security (login form + BCrypt) |
| Banco de dados | H2 (arquivo persistente, único perfil configurado no projeto) |
| Documentação | Swagger / OpenAPI |

## Perfis de Usuário

| Perfil | Acesso |
|---|---|
| **TUTOR** | Vê e gerencia apenas os próprios pets; pode agendar consultas |
| **VET** | Vê todos os pets, clínicas e agendamentos; pode registrar vacinas aplicadas |

## Como Executar

```bash
mvn spring-boot:run
```

- **Aplicação web**: http://localhost:8080/login
- **Swagger (API REST)**: http://localhost:8080/swagger-ui.html
- **Console H2**: http://localhost:8080/h2-console (JDBC URL: `jdbc:h2:file:./data/vetflowdb`)

### Usuários de teste (senha: `senha123` para todos)

| Perfil | E-mail |
|---|---|
| Veterinário | vet@vetflow.com |
| Tutor | carlos.mendes@email.com |

Também é possível criar uma nova conta de tutor pela tela **Cadastre-se** no login.

## Controle de Versão do Banco (Flyway)

O schema não é mais gerado automaticamente pelo Hibernate (`ddl-auto=create-drop`). Agora o Flyway é a única fonte de verdade:

| Migration | Conteúdo |
|---|---|
| `V1__base_schema.sql` | Tabelas de domínio (tutores, pets, clínicas, agendamentos, vacinas, medicamentos) |
| `V2__create_users_table.sql` | Tabela de usuários (`vf_users`) para autenticação |
| `V3__seed_data.sql` | Carga inicial de dados de demonstração |

Toda alteração futura de schema deve entrar como uma nova migration versionada (`V4__...`, `V5__...`), nunca alterando as anteriores.

## Fluxos Completos (além do CRUD)

### 1. Agendamento de consulta (Tutor)
O tutor acessa o pet, clica em **Agendar consulta**, escolhe clínica, tipo e data/hora, confirma — o sistema cria o agendamento e retorna ao histórico do pet.

`GET/POST /web/appointments/schedule/{petId}`

### 2. Registro de vacina aplicada (Veterinário)
O veterinário acessa o pet, clica em **Registrar vacina**, preenche nome, data de aplicação e próxima dose, confirma — o sistema salva a vacina no histórico do pet. Rota restrita ao perfil `VET`.

`GET/POST /web/vaccines/apply/{petId}`

## Endpoints REST (API)

*(inalterados desde a sprint anterior — protegidos por autenticação)*

### Tutors `/api/tutors`, Pets `/api/pets`, Clinics `/api/clinics`, Appointments `/api/appointments`, Vaccines `/api/vaccines`, Medications `/api/medications`

Ver documentação completa e testável no Swagger: `/swagger-ui.html`

## Validações

- Formulário de cadastro de tutor: nome, e-mail (formato válido), telefone e senha (mínimo 6 caracteres) obrigatórios.
- Formulário de agendamento: clínica, tipo e data/hora obrigatórios; tutor só agenda para os próprios pets.
- Formulário de vacina: nome, data de aplicação e próxima dose obrigatórios; próxima dose deve ser posterior à aplicação.

## Benefícios para o Negócio

- Aumento da recorrência de consultas preventivas nas clínicas parceiras
- Redução de vacinas vencidas e abandono de tratamentos
- Histórico longitudinal estruturado por pet
- Acesso segmentado por perfil, reduzindo risco de acesso indevido a dados de outros tutores