
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
import requests
from odoo import models, fields, api

class CommunityMember(models.Model):
    _name = 'community.member'
    _description = 'Membros da Comunidade'

    name = fields.Char(string='Nome', required=True)
    email = fields.Char(string='Email', required=True)
    points = fields.Integer(string='Pontos', default=0)
    level = fields.Char(string='Nível', compute='_compute_level')

    @api.depends('points')
    def _compute_level(self):
        for member in self:
            if member.points >= 1000:
                member.level = 'Expert'
            elif member.points >= 500:
                member.level = 'Advanced'
            else:
                member.level = 'Beginner'

class CommunityQwenChat(models.Model):
    _name = 'community.qwen.chat'
    _description = 'Chatbot Qwen para Comunidade'

    question = fields.Text(string='Pergunta')
    answer = fields.Text(string='Resposta')

    def call_qwen_api(self):
        api_key = self.env['ir.config_parameter'].sudo().get_param('qwen.api_key')  # Armazene sua API Key nas configurações
        endpoint = 'https://api.qwen.com/v1/chat'

        try:
            response = requests.post(endpoint, json={
                "prompt": self.question,
                "max_tokens": 50,
                "temperature": 0.7
            }, headers={
                "Content-Type": "application/json",
                "Authorization": f"Bearer {api_key}"
            })

            if response.status_code == 200:
                self.answer = response.json().get("answer", "Erro ao processar a resposta.")
            else:
                self.answer = "Erro na chamada à API."
        except Exception as e:
            self.answer = str(e)
