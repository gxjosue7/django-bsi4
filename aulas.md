📘 Aula 20 — Primeiro endpoint com Django REST Framework
Objetivo

Criar o primeiro endpoint REST completo, já com CRUD, usando ModelSerializer, ModelViewSet e DefaultRouter.

1. Instalando e configurando o DRF

Instale o pacote somente agora:

uv add djangorestframework
O aplicativo produtos já foi adicionado na Aula 17. Agora, acrescente apenas "rest_framework" ao INSTALLED_APPS, preservando os aplicativos e as demais configurações que o Django já criou. Se o bloco REST_FRAMEWORK ainda não existir, crie-o; se já existir, acrescente as configurações indicadas ao bloco existente.

"rest_framework",
2. Criando o ModelSerializer

Crie produtos/serializers.py:

from rest_framework import serializers

from .models import Produto


class ProdutoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Produto
        fields = ("id", "nome", "preco")
O ModelSerializer representa instâncias como dados que podem virar JSON e valida entradas antes de salvar. Ele também aproveita os tipos e limites definidos no Model.

3. Criando a ModelViewSet

Substitua produtos/views.py por:

from rest_framework.viewsets import ModelViewSet

from .models import Produto
from .serializers import ProdutoSerializer


class ProdutoViewSet(ModelViewSet):
    queryset = Produto.objects.all()
    serializer_class = ProdutoSerializer
A ModelViewSet disponibiliza automaticamente as ações de listagem, detalhe, criação, atualização completa (PUT), atualização parcial (PATCH) e exclusão. PATCH representa atualização parcial. Nesta etapa didática, os testes do contrato utilizam PUT para atualização completa; o PATCH existe porque faz parte das ações padrão da ModelViewSet, mas não será explorado neste momento.

4. Registrando o DefaultRouter

Substitua config/urls.py por:

from django.contrib import admin
from django.urls import include, path
from rest_framework.routers import DefaultRouter

from produtos.views import ProdutoViewSet

router = DefaultRouter()
router.register("produtos", ProdutoViewSet, basename="produto")

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include(router.urls)),
]
O Router gera as URLs a partir do registro da ViewSet. O padrão é:

Model
  ↓
ModelSerializer
  ↓
ModelViewSet
  ↓
DefaultRouter
  ↓
Endpoint REST
A responsabilidade de cada camada é diferente: o Model cuida da persistência e do ORM; o Serializer cuida da representação e da validação; a ViewSet reúne as ações da API; e o Router gera as URLs.

5. Testando o CRUD

GET    /api/produtos/
GET    /api/produtos/{id}/
POST   /api/produtos/
PUT    /api/produtos/{id}/
PATCH  /api/produtos/{id}/
DELETE /api/produtos/{id}/
Use o navegador para o GET e a interface navegável do DRF para testar criação, atualização parcial, atualização completa e exclusão. Ao final, a API funciona, mas ainda não possui documentação OpenAPI configurada.

💾 Commit sugerido

Depois de testar as seis operações do CRUD:

git add .
git commit -m "feat(drf): cria primeiro endpoint de produtos"
📘 Aula 21 — OpenAPI e Swagger
Objetivo

Adicionar documentação automática à API existente sem alterar sua arquitetura.

1. Instalando e configurando o drf-spectacular

uv add drf-spectacular
Em config/settings.py, acrescente o aplicativo drf_spectacular a INSTALLED_APPS e acrescente as configurações do schema. Preserve os aplicativos padrão, rest_framework e as demais configurações já existentes; não substitua todo o arquivo settings.py. Se REST_FRAMEWORK já existir, acrescente DEFAULT_SCHEMA_CLASS ao bloco sem apagar outras chaves:

"drf_spectacular",

REST_FRAMEWORK = {
  # acrescente esta chave ao bloco existente, se ele já existir
    "DEFAULT_SCHEMA_CLASS": "drf_spectacular.openapi.AutoSchema",
}

SPECTACULAR_SETTINGS = {
    "TITLE": "API de Produtos",
    "DESCRIPTION": "API de produtos construída com Django REST Framework",
    "VERSION": "1.0.0",
}
2. Criando as URLs da documentação

Atualize config/urls.py, preservando o Router da Aula 20:

from drf_spectacular.views import (
    SpectacularAPIView,
    SpectacularRedocView,
    SpectacularSwaggerView,
)

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include(router.urls)),
    path("api/schema/", SpectacularAPIView.as_view(), name="schema"),
    path("api/docs/", SpectacularSwaggerView.as_view(url_name="schema"), name="swagger-ui"),
    path("api/redoc/", SpectacularRedocView.as_view(url_name="schema"), name="redoc"),
]
Abra /api/schema/ para o documento OpenAPI, /api/docs/ para o Swagger UI e /api/redoc/ para o Redoc. O Swagger deve documentar o CRUD completo criado na Aula 20.

Aula 20: a API funciona. Aula 21: a API funciona e também tem documentação automática. Nenhuma ViewSet é criada ou substituída nesta aula.

💾 Commit sugerido

Depois de abrir /api/docs/ e confirmar todas as operações:

git add .
git commit -m "docs(api): adiciona documentação OpenAPI e Swagger"
📘 Aula 22 — Validações
Objetivo

Adicionar regras de negócio ao ProdutoSerializer e confirmar que a validação acontece antes da persistência.

Atualize produtos/serializers.py:

from decimal import Decimal

from rest_framework import serializers

from .models import Produto


class ProdutoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Produto
        fields = ("id", "nome", "preco")

    def validate_nome(self, value):
        nome_limpo = value.strip()
        if len(nome_limpo) < 2:
            raise serializers.ValidationError(
                "O nome deve possuir pelo menos 2 caracteres."
            )
        return nome_limpo

    def validate_preco(self, value):
        if value <= Decimal("0"):
            raise serializers.ValidationError("O preço deve ser maior que zero.")
        return value
DecimalField entrega um Decimal, por isso a comparação usa Decimal("0"), sem converter o valor monetário para float.

No Swagger, envie nome: "A" e preco: 0: a resposta deve ser 400 Bad Request. Depois envie {"nome": "Cabo USB", "preco": 20.00} e confirme 201 Created. O objeto só é salvo depois que todas as validações passam.

💾 Commit sugerido

Depois de testar casos válidos e inválidos no Swagger:

git add .
git commit -m "feat(api): adiciona validações de produtos"
📘 Aula 23 — Filtros
Objetivo

Filtrar produtos por faixa de preço usando django-filter.

1. Instalando e configurando

uv add django-filter
Acrescente django_filters a INSTALLED_APPS. No REST_FRAMEWORK, preserve DEFAULT_SCHEMA_CLASS e acrescente:

"DEFAULT_FILTER_BACKENDS": [
    "django_filters.rest_framework.DjangoFilterBackend",
],
Crie produtos/filters.py:

from django_filters import rest_framework as filters

from .models import Produto


class ProdutoFilter(filters.FilterSet):
    preco_minimo = filters.NumberFilter(field_name="preco", lookup_expr="gte")
    preco_maximo = filters.NumberFilter(field_name="preco", lookup_expr="lte")

    class Meta:
        model = Produto
        fields = ("preco_minimo", "preco_maximo")
Atualize produtos/views.py:

from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.viewsets import ModelViewSet

from .filters import ProdutoFilter
from .models import Produto
from .serializers import ProdutoSerializer


class ProdutoViewSet(ModelViewSet):
    queryset = Produto.objects.all()
    serializer_class = ProdutoSerializer
    filter_backends = (DjangoFilterBackend,)
    filterset_class = ProdutoFilter
gte significa maior ou igual e lte significa menor ou igual. Teste:

GET /api/produtos/?preco_minimo=100&preco_maximo=1000
GET /api/produtos/?preco_minimo=100
GET /api/produtos/?preco_maximo=1000
Confira no Swagger que os parâmetros aparecem no endpoint e teste também uma combinação sem resultados.

💾 Commit sugerido

Depois de testar os dois limites e o intervalo:

git add .
git commit -m "feat(api): adiciona filtros por preço"
📘 Aula 24 — Ordenação e busca textual
Objetivo

Combinar filtros, ordenação e busca textual no mesmo endpoint, mantendo somente os campos existentes nesta etapa.

Atualize produtos/views.py:

from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import OrderingFilter, SearchFilter
from rest_framework.viewsets import ModelViewSet

from .filters import ProdutoFilter
from .models import Produto
from .serializers import ProdutoSerializer


class ProdutoViewSet(ModelViewSet):
    queryset = Produto.objects.all()
    serializer_class = ProdutoSerializer
    filter_backends = (DjangoFilterBackend, SearchFilter, OrderingFilter)
    filterset_class = ProdutoFilter
    ordering_fields = ("nome", "preco")
    ordering = ("id",)
    search_fields = ("nome",)
OrderingFilter aceita ordering=nome, ordering=-nome, ordering=preco e ordering=-preco. SearchFilter faz busca parcial e sem diferenciar maiúsculas em nome.

GET /api/produtos/?ordering=nome
GET /api/produtos/?ordering=-preco
GET /api/produtos/?search=mouse
GET /api/produtos/?preco_minimo=100&search=teclado&ordering=nome
Os três mecanismos se combinam no queryset: filtros estruturados, busca, ordenação e, na aula seguinte, paginação. marca e descricao entrarão na busca na Aula 26; estoque ficará fora por ser numérico.

💾 Commit sugerido

Depois de conferir consultas isoladas e combinadas no Swagger:

git add .
git commit -m "feat(api): adiciona busca e ordenação"
📘 Aula 25 — Paginação
Objetivo

Preservar no Django o contrato de paginação usado nas Partes Express/FastAPI:

{
  "page": 1,
  "page_size": 10,
  "total_pages": 5,
  "results": []
}
O PageNumberPagination padrão do DRF usa count, next, previous e results. A classe abaixo continua usando PageNumberPagination, mas reformata a resposta e retorna uma lista vazia quando a página solicitada está além do limite.

Crie produtos/pagination.py:

Normalmente, não é necessário sobrescrever paginate_queryset() para usar paginação no DRF. Esta sobrescrita existe especificamente neste tutorial para preservar o contrato equivalente ao construído anteriormente com Express e FastAPI: quando o cliente solicita uma página além da última, a API deve retornar 200 com results: [], em vez do erro de página inválida que o comportamento padrão pode produzir.

from math import ceil

from rest_framework.exceptions import ValidationError
from rest_framework.pagination import PageNumberPagination
from rest_framework.response import Response


class ProdutoPagination(PageNumberPagination):
    page_size_query_param = "page_size"
    max_page_size = 100

    def paginate_queryset(self, queryset, request, view=None):
        self.request = request
    page_size_param = request.query_params.get(self.page_size_query_param)
    if page_size_param is not None:
      try:
        if int(page_size_param) > self.max_page_size:
          raise ValidationError({
            "page_size": "O campo page_size não pode passar de 100."
          })
      except ValueError:
        pass

        page_size = self.get_page_size(request)
        if page_size is None:
            return None

        self.paginator = self.django_paginator_class(queryset, page_size)
        try:
            self.page_number = int(request.query_params.get(self.page_query_param, 1))
        except (TypeError, ValueError):
            raise ValidationError({"page": "Informe um número inteiro positivo."})

        if self.page_number < 1:
            raise ValidationError({"page": "Informe um número inteiro positivo."})

        self.total_pages = ceil(self.paginator.count / page_size)
        if self.total_pages == 0 or self.page_number > self.total_pages:
            self.page = None
            return []

        self.page = self.paginator.page(self.page_number)
        return list(self.page)

    def get_paginated_response(self, data):
        return Response({
            "page": self.page_number,
            "page_size": self.paginator.per_page,
            "total_pages": self.total_pages,
            "results": data,
        })
A classe respeita page_size, rejeita valores acima de 100 com 400 e detail, valida páginas menores que 1 e calcula total_pages antes do corte. Assim, uma página além do limite produz 200 com results: [], como nas Partes 1–6.

Configurando globalmente

Em config/settings.py, acrescente estas chaves ao bloco REST_FRAMEWORK existente, preservando as demais configurações. Se o bloco ainda não existir, crie-o:

"DEFAULT_PAGINATION_CLASS": "produtos.pagination.ProdutoPagination",
"PAGE_SIZE": 10,
Testando

GET /api/produtos/?page=1
GET /api/produtos/?page=2&page_size=20
GET /api/produtos/?search=mouse&ordering=-preco&page=1&page_size=5
GET /api/produtos/?page=999
A última requisição deve retornar 200 com results vazia. Teste também page_size=101, que deve retornar 400 com detail; valores até 100 seguem normalmente. Filtros, busca e ordenação são aplicados ao queryset antes da paginação, exatamente como no contrato anterior.

💾 Commit sugerido

Depois de verificar o formato da resposta, o limite de tamanho e as combinações:

git add .
git commit -m "feat(api): adiciona paginação customizada"
📘 Aula 26 — Exercício: Evoluindo o Produto
Objetivo

Repetir no Django a evolução feita nas Aulas 14–16 e perceber que uma mudança no Model atravessa todas as camadas:

Model
→ migration
→ serializer
→ validação
→ filtro
→ ordenação
→ busca
→ Swagger
→ teste
Os exercícios são cumulativos. Depois de cada campo, gere e aplique a migration, atualize o código, confira o Swagger e execute os testes.

1. Marca

marca será obrigatório, terá de 2 a 50 caracteres e participará do filtro exato, da ordenação e da busca.

Desafio: adicione o campo ao Model, ao Serializer e ao ProdutoFilter, e atualize ordering_fields e search_fields.

Ver solução
Teste POST válido, POST com marca curta, GET /api/produtos/?marca=dell, ?ordering=marca e ?search=dell.

2. Estoque

estoque será inteiro, obrigatório, não negativo, filtrável por intervalo e ordenável. Ele não participará da busca textual.

Desafio: adicione o campo, gere a migration e atualize serializer, validação, filtro e ordenação.

Ver solução
Teste estoque=0 (válido), -1, "dez", 5.5, ?estoque_minimo=10&estoque_maximo=30 e ?ordering=-estoque. O campo inteiro do Model e o campo gerado pelo ModelSerializer rejeitam tipos incompatíveis antes da persistência.

3. Descrição

descricao será opcional, aceitará null ou texto vazio, terá no máximo 500 caracteres, será ordenável e participará da busca.

Desafio: adicione o campo e atualize serializer, validação, ordenação e busca.

Ver solução
Teste POST sem descrição, texto acima de 500 caracteres, ?search=usb-c e ?ordering=descricao. Depois de cada campo, confirme no Swagger que o schema, os parâmetros e as operações do CRUD continuam atualizados.

💾 Commit sugerido

Depois de concluir os três campos e testar o ciclo completo:

git add .
git commit -m "feat(produtos): adiciona marca estoque e descricao"
A Parte 7 termina com o mesmo conceito das Partes 1–6, mas com outra implementação: o Express e o FastAPI resolvem várias etapas manualmente; o Django combina ORM, migrations, serializers, ViewSets, routers e filter backends para declarar o mesmo contrato /api/produtos/.