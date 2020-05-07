from django.shortcuts import render,redirect
from webtest.models import GoodsInfo, TypeInfo
from usertest.models import UserInfo
from carttest.models import CartInfo
from django.http import JsonResponse,HttpResponse
from django.core.paginator import Paginator
from comment.forms import CommentForm

# Create your views here.

def index(request):
    type_list = TypeInfo.objects.all()  # 获取所有分类
    iscart = 1
    user = ''
    if request.COOKIES.get('isLogin'):
        loginName = request.session.get('loginName')

        if loginName:
            user = UserInfo.objects.get(uname=loginName)

    list1 = []

    for type_info in type_list:
        new = type_info.goodsinfo_set.order_by('-id')[0:4]
        fire = type_info.goodsinfo_set.order_by('-gclick')[0:3]
        list1.append({'type': type_info, 'new': new, 'fire': fire})
    return render(request, 'good/index.html', {'title': '首页',
                                               'iscart': iscart,
                                               'list': list1,
                                               'user':user})

def isExit(request):
    res = redirect('/')
    res.set_cookie('isLogin', '', max_age=-1)
    request.session['loginName'] = ''

    return res

def detail(request, id):

    good = GoodsInfo.objects.get(id=id)
    good.gclick += 1 #点击加一
    good.save()

    form = CommentForm() #创建表单对象
    comment_list = good.comment_set.all()
    gnew = good.gParent.goodsinfo_set.order_by('-id')[0:2]
    res = render(request, 'good/detail.html', {'title': '商品详情', 'iscart': 1,
                                               'good': good, 'new': gnew,
                                               'form':form , 'comment_list':comment_list})

    if request.COOKIES.get('isLogin'):
        jin = request.COOKIES.get('jin', '[]')
        jin_list = eval(jin)
        if jin:
            jin_list.insert(0, id)
            if len(jin_list) > 5:
                jin_list.pop()
        res.set_cookie('jin', str(jin_list))

    return res


def list(request, id,page_num,order_str):
    user = ''
    if request.COOKIES.get('isLogin'):
        name = request.session.get('loginName')
        user = UserInfo.objects.get(uname=name)


    type = TypeInfo.objects.get(id=id)
    order = '-id'
    if order_str == '1':
        order = '-gprice'
    elif order_str == '2':
        order = '-gclick'

    goods = type.goodsinfo_set.order_by(order)

    p = Paginator(goods,10)
    page_num = int(page_num)
    page = p.page(page_num)
    list1 = [i for i in range(max(page_num - 2, 1), page_num)]
    list2 = [i for i in range(page_num, min(page_num + 2, p.num_pages + 1))]

    page_range = list1 + list2

    print(1)

    # 加上首页
    if page_range[0] - 1 >= 1:
        page_range.insert(0, 1)

    # 加上尾页
    if page_range[-1] <= p.num_pages - 1:
        page_range.append(p.num_pages)

    # 给隐藏的的页码补上 ‘...’
    if page_num - 2 > 2:
        page_range.insert(1, '...')
    # 给后边隐藏的的页码补上 ‘...’
    if page_num + 2 < p.num_pages:
        page_range.insert(-1, '...')


    gnew = type.goodsinfo_set.order_by('-id')[0:2]

    dict1={'title': '商品列表',
           'iscart': 1,
           'type': type,
           'new': gnew,
           'page':page,
           'tid':id,
           'order_str':order_str,
           'page_range':page_range,
           'user':user
           }

    return render(request, 'good/list.html',dict1 )
