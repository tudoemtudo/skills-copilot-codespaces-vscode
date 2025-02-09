
{
    'name': 'Community with Qwen Integration',
    'version': '1.0',
    'summary': 'Plataforma de comunidade com integração ao Qwen 2.5-Max',
    'description': '''
        Este módulo cria uma plataforma de comunidade no Odoo com recursos como:
        - Fórum de discussão
        - Gamificação
        - Chatbot interativo usando a API do Qwen
        - Integração com GitHub
    ''',
    'author': 'Seu Nome',
    'depends': ['base', 'website', 'gamification', 'forum'],
    'data': [
        'views/community_qwen_views.xml',
        'views/website_templates.xml',
    ],
    'assets': {
        'web.assets_frontend': [
            'community_qwen/static/src/css/styles.css',
            'community_qwen/static/src/js/chatbot.js',
        ],
    },
    'installable': True,
    'application': True,
}
